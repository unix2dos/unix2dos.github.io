# 1. 基础

### 1.1 进程的内存

程序运行进程的总大小可以超过实际可用的物理内存的大小。每个进程都可以有自己独立的虚拟地址空间。然后通过CPU和MMU把虚拟内存地址转换为实际物理地址。

![0](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/0.png)



最高位的1GB是linux内核空间，用户代码不能写，否则触发段错误。下面的3GB是进程使用的内存。

+ Kernel space：linux内核空间内存
+ Stack：进程栈空间，程序运行时使用。它向下增长，系统自动管理
+ Memory Mapping Segment：内存映射区，通过mmap系统调用，将文件映射到进程的地址空间，或者匿名映射。
+ Heap：堆空间。这个就是程序里动态分配的空间。linux下使用malloc调用扩展（用brk/sbrk扩展内存空间），free函数释放（也就是缩减内存空间）
+ BSS段：包含未初始化的静态变量和全局变量
+ Data段：代码里已初始化的静态变量、全局变量
+ Text段：代码段，进程的可执行文件



### 1.2 golang逃逸分析

C/C++，我们使用 malloc 或者 new 申请的变量会被分配到堆上。但是 Golang 并不是这样，虽然 Golang 语言里面也有 new。Golang 编译器决定变量应该分配到什么地方时会进行逃逸分析。下面是一个简单的例子。

```go
package main

import ()

func foo() *int {
    var x int
    return &x
}

func bar() int {
    x := new(int)
    *x = 1
    return *x
}

func main() {}
```

将上面文件保存为 main.go，执行下面命令

```bash
$ go run -gcflags '-m -l' main.go

# command-line-arguments
./main.go:4:6: moved to heap: x
./main.go:9:10: new(int) does not escape
```


上面的意思是 foo() 中的 x 最后在堆上分配，而 bar() 中的 x 最后分配在了栈上。


如何得知变量是分配在栈（stack）上还是堆（heap）上？准确地说，你并不需要知道。Golang 中的变量只要被引用就一直会存活，存储在堆上还是栈上由内部实现决定而和具体的语法没有关系。



# 2. 内存分配数据结构

Golang运行时的内存分配算法主要源自 Google 为 C 语言开发的 **TCMalloc** 算法，全称Thread-Caching Malloc。

核心思想就是把内存分为多级管理，从而降低锁的粒度。它将可用的堆内存采用二级分配的方式进行管理：每个线程都会自行维护一个独立的内存池，进行内存分配时优先从该内存池中分配，当内存池不足时才会向全局内存池申请，以避免不同线程对全局内存池的频繁竞争。

![1](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/1.png)



### 2.1 mspan

mspan它是golang内存管理中的基本单位，也是由页组成的。一个页的大小是:   ?

mspan里面按照8*2n大小（8b，16b，32b .... ），每一个mspan又分为多个object。mspan中的m应该是memory的第一个字母。

```go
type mspan struct {
    next *mspan     // next span in list, or nil if none
    prev *mspan     // previous span in list, or nil if none
    list *mSpanList // For debugging. TODO: Remove.

    startAddr     uintptr   // address of first byte of span aka s.base()
    npages        uintptr   // number of pages in span
    stackfreelist gclinkptr // list of free stacks, avoids overloading freelist
    // freeindex is the slot index between 0 and nelems at which to begin scanning
    // for the next free object in this span.
    freeindex uintptr
    // TODO: Look up nelems from sizeclass and remove this field if it
    // helps performance.
    nelems uintptr // number of object in the span.
    ...
    // 用位图来管理可用的 free object，1 表示可用
    allocCache uint64
    
    ...
    sizeclass   uint8      // size class
    ...
    elemsize    uintptr    // computed from sizeclass or from npages
    ...
}
```

解释:

```bash
next, prev: 指针域，因为 mspan 一般都是以链表形式使用。
npages: mspan 的大小为 page 大小的整数倍。
sizeclass: 0 ~ _NumSizeClasses 之间的一个值。比如，sizeclass = 3，那么这个 mspan 被分割成 32 byte 的块。
elemsize: elemsize = page_size * npages。
```



### 2.2 mcache

mcache可以为golang中每个Processor提供内存cache使用，每一个mcache的组成单位也是mspan。

我们知道每个 Gorontine 的运行都是绑定到一个 P 上面，mcache 是每个 P 的 cache。这么做的好处是分配内存时不需要加锁。

```go
// Per-thread (in Go, per-P) cache for small objects.
// No locking needed because it is per-thread (per-P).
type mcache struct {
    // The following members are accessed on every malloc,
    // so they are grouped here for better caching.
    next_sample int32   // trigger heap sample after allocating this many bytes
    local_scan  uintptr // bytes of scannable heap allocated

    // 小对象分配器，小于 16 byte 的小对象都会通过 tiny 来分配。
    tiny             uintptr
    tinyoffset       uintptr
    local_tinyallocs uintptr // number of tiny allocs not counted in other stats

    // The rest is not accessed on every malloc.
    alloc [_NumSizeClasses]*mspan // spans to allocate from

    stackcache [_NumStackOrders]stackfreelist

    // Local allocator stats, flushed during GC.
    local_nlookup    uintptr                  // number of pointer lookups
    local_largefree  uintptr                  // bytes freed for large objects (>maxsmallsize)
    local_nlargefree uintptr                  // number of frees for large objects (>maxsmallsize)
    local_nsmallfree [_NumSizeClasses]uintptr // number of frees for small objects (<=maxsmallsize)
}
```

解释:
```bash
alloc [_NumSizeClasses]*mspan
_NumSizeClasses = 67, 每个数组元素用来包含特定大小的块。67 种块大小为 0，8 byte, 16 byte....

var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
```



### 2.3 mcentral

当mcache中空间不够用，可以向mcentral申请内存。可以理解为mcentral为mcache的一个“缓存库”，供mcaceh使用。它的内存组成单位也是mspan。
mcentral里有两个双向链表，一个链表表示还有空闲的mspan待分配，一个表示链表里的mspan都被分配了。

```go
type mcentral struct {
    lock      mutex
    sizeclass int32
    nonempty  mSpanList // list of spans with a free object, ie a nonempty free list
    empty     mSpanList // list of spans with no free objects (or cached in an mcache)
}

type mSpanList struct {
    first *mspan
    last  *mspan
}

```

解释

```bash
sizeclass: 0 ~ _NumSizeClasses 之间的一个值。
lock: 因为会有多个 P 过来竞争。
nonempty: mspan 的双向链表，当前 mcentral 中可用的 mspan list。
empty: 已经被使用的，可以认为是一种对所有 mspan 的 track。
```

mcentral 存在于什么地方？ mcentral 和 mheap 作为全局的结构，这两部分是可以定义在一起的。实际上也是这样，mcentral 包含在 mheap 中。



### 2.4 mheap

mheap负责大内存的分配。当mcentral内存不够时，可以向mheap申请。那mheap没有内存资源呢？跟tcmalloc一样，向OS操作系统申请。
还有，大于32KB的内存，也是直接向mheap申请。

```go
type mheap struct {
    lock      mutex
    free      [_MaxMHeapList]mSpanList // free lists of given length
    freelarge mSpanList                // free lists length >= _MaxMHeapList
    busy      [_MaxMHeapList]mSpanList // busy lists of large objects of given length
    busylarge mSpanList                // busy lists of large objects length >= _MaxMHeapList
    sweepgen  uint32                   // sweep generation, see comment in mspan
    sweepdone uint32                   // all spans are swept

   
    allspans []*mspan // all spans out there
    spans []*mspan
    sweepSpans [2]gcSweepBuf

    _ uint32 // align uint64 fields on 32-bit for atomics

    // Proportional sweep
    pagesInUse        uint64  // pages of spans in stats _MSpanInUse; R/W with mheap.lock
    spanBytesAlloc    uint64  // bytes of spans allocated this cycle; updated atomically
    pagesSwept        uint64  // pages swept this cycle; updated atomically
    sweepPagesPerByte float64 // proportional sweep ratio; written with lock, read without
    // TODO(austin): pagesInUse should be a uintptr, but the 386
    // compiler can't 8-byte align fields.

    // Malloc stats.
    largefree  uint64                  // bytes freed for large objects (>maxsmallsize)
    nlargefree uint64                  // number of frees for large objects (>maxsmallsize)
    nsmallfree [_NumSizeClasses]uint64 // number of frees for small objects (<=maxsmallsize)

    // range of addresses we might see in the heap
    bitmap         uintptr // Points to one byte past the end of the bitmap
    bitmap_mapped  uintptr
    arena_start    uintptr
    arena_used     uintptr // always mHeap_Map{Bits,Spans} before updating
    arena_end      uintptr
    arena_reserved bool

    // central free lists for small size classes.
    // the padding makes sure that the MCentrals are
    // spaced CacheLineSize bytes apart, so that each MCentral.lock
    // gets its own cache line.
    central [_NumSizeClasses]struct {
        mcentral mcentral
        pad      [sys.CacheLineSize]byte
    }

    spanalloc             fixalloc // allocator for span*
    cachealloc            fixalloc // allocator for mcache*
    specialfinalizeralloc fixalloc // allocator for specialfinalizer*
    specialprofilealloc   fixalloc // allocator for specialprofile*
    speciallock           mutex    // lock for special record allocators.
}

var mheap_ mheap  // mheap_ 是一个全局变量，会在系统初始化的时候初始化
```

解释

```bash
allspans []*mspan: 所有的 spans 都是通过 mheap_ 申请，所有申请过的 mspan 都会记录在 allspans。结构体中的 lock 就是用来保证并发安全的。

sweepgen, sweepdone: GC 相关。（Golang 的 GC 策略是 Mark & Sweep, 这里是用来表示 sweep 的)
```



# 3. 内存分配

1. object size > 32K，则使用 mheap 直接分配。
2. object size < 16 byte，使用 mcache 的小对象分配器 tiny 直接分配。 
3. object size > 16 byte && size <=32K byte 时，先使用 mcache 中对应的 size class 分配。
4. 如果 mcache 对应的 size class 的 span 已经没有可用的块，则向 mcentral 请求。
5. 如果 mcentral 也没有可用的块，则向 mheap 申请，并切分。
6. 如果 mheap 也没有合适的 span，则想操作系统申请。



# 4. 参考资料

+ https://www.cnblogs.com/jiujuan/p/13922551.html

+ http://legendtkl.com/2017/04/02/golang-alloc/




https://juejin.cn/post/6844903795739082760





