---
title: "golang内存管理"
date: 2021-01-20 00:02:00
tags:
- golang
---

# 1. 基础

### 1.1 进程的内存

程序运行进程的总大小可以超过实际可用的物理内存的大小。每个进程都可以有自己独立的虚拟地址空间。然后通过CPU和MMU把虚拟内存地址转换为实际物理地址。

![0](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/1.png)

<!-- more -->

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



### 1.3 golang内存结构

Go程序启动的时候，会将“虚拟内存”按照自己的方式，切成小块后管理。申请到的内存块被分配了三个区域，在X64上分别是512MB，16GB，512GB大小。

![heap-before-go-1-10](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/2.png)

`spans区域`存放`mspan`的指针。

`bitmap区域`标识`arena`区域哪些地址保存了对象，并且用`4bit`标志位表示对象是否包含指针、`GC`标记信息。

`arena区域`就是我们所谓的堆区，Go动态分配的内存都是在这个区域，它把内存分割成`8KB`大小的页，一些页组合起来称为`mspan`。



### 1.4 golang内存管理单元mspan

Go默认采用8192B(8KB)大小的页，页的粒度保持为8KB。

但是有的变量很小就是数字，有的却是一个复杂的结构体，所以基于TCMalloc模型的Go还将内存页分为67个不同大小级别，从8字节到32KB分了67 种( 8 byte, 16 byte....32KB）。

例如下图, 将8 KB页 划分为1KB的大小等级。

![img{512x368}](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/3.png)



`mspan`：Go中内存管理的基本单元，是一个双向链表对象，其中包含页面的起始地址，它具有的页面的span类以及它包含的页面数。

```go
// path: /usr/local/go/src/runtime/mheap.go

type mspan struct {
    //链表前向指针，用于将span链接起来
    next *mspan 
    //链表前向指针，用于将span链接起来
    prev *mspan 
    // 起始地址，也即所管理页的地址
    startAddr uintptr 
    // 管理的页数
    npages uintptr 
    // 块个数，表示有多少个块可供分配
    nelems uintptr 

    //分配位图，每一位代表一个块是否已分配
    allocBits *gcBits 

    // 已分配块的个数
    allocCount uint16 
    // class表中的class ID，和Size Classs相关
    spanclass spanClass  

    // class表中的对象大小，也即块大小
    elemsize uintptr 
}
```



# 2. 内存管理组件

Golang运行时的内存分配算法主要源自 Google 为 C 语言开发的 **TCMalloc** 算法，全称Thread-Caching Malloc。

核心思想就是把内存分为多级管理，从而降低锁的粒度。它将可用的堆内存采用二级分配的方式进行管理：每个线程都会自行维护一个独立的内存池，进行内存分配时优先从该内存池中分配，当内存池不足时才会向全局内存池申请，以避免不同线程对全局内存池的频繁竞争。

![1](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/4.png)



### 2.1 mcache

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
_NumSizeClasses = 67, 每个数组元素用来包含特定大小的块。67 种块大小为 0，8 byte, 16 byte....32KB

var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
```



例如: 申请32字节内存,  32b的这种`mspan`能满足需求，那么分配内存的时候就会给它分配一个32字节大小的`mspan`。

![11](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/5.png)

### 2.2 mcentral

当工作线程的`mcache`中没有合适（也就是特定大小的）的`mspan`时就会从`mcentral` 去获取。`mcentral`被所有的工作线程共同享有，存在多个`goroutine`竞争的情况，因此从`mcentral`获取资源时需要加锁。

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

`mcentral`里维护着两个双向链表，**nonempty**表示链表里还有空闲的`mspan`待分配。**empty**表示这条链表里的`mspan`都被分配了`object`。

![12](golang%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/6.png)

`mcache`从`mcentral`获取和归还`mspan`的流程：

- 获取 加锁；从`nonempty`链表找到一个可用的`mspan`；并将其从`nonempty`链表删除；将取出的`mspan`加入到`empty`链表；将`mspan`返回给工作线程；解锁。
- 归还 加锁；将`mspan`从`empty`链表删除；将`mspan`加入到`nonempty`链表；解锁。



### 2.3 mheap

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

`mheap`里的`arena` 区域是真正的堆区，运行时会将 `8KB` 看做一页，这些内存页中存储了所有在堆上初始化的对象。



# 3. 内存分配

### 3.1 分配策略

- Go在程序启动时，会向操作系统申请一大块内存，由`mheap`结构全局管理。
- Go内存管理的基本单元是`mspan`，每种`mspan`可以分配特定大小的`object`。
- `mcache`, `mcentral`, `mheap`是`Go`内存管理的三大组件，`mcache`管理线程在本地缓存的`mspan`；`mcentral`管理全局的`mspan`供所有线程使用；`mheap`管理`Go`的所有动态分配内存。
- object size < 16 byte，使用 mcache 的小对象分配器 tiny 直接分配。 object size > 16 byte && size <=32K byte 时，先使用 mcache 中对应的 size class 分配。object size > 32K，则使用 mheap 直接分配。
- 如果 mcache 对应的 size class 的 span 已经没有可用的块，则向 mcentral 请求。如果 mcentral 也没有可用的块，则向 mheap 申请。如果 mheap 也没有合适的 span，则想操作系统申请。



# 4. 参考资料

+ https://www.cnblogs.com/jiujuan/p/13922551.html
+ http://legendtkl.com/2017/04/02/golang-alloc/
+ https://zhuanlan.zhihu.com/p/352133292
+ https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/



