channel

func makeThumbnails6(filenames <-chan string) int64 {
    sizes := make(chan int64)
    var wg sync.WaitGroup // number of working goroutines
    for f := range filenames {
        wg.Add(1)
        // worker
        go func(f string) {
            defer wg.Done()
            thumb, err := thumbnail.ImageFile(f)
            if err != nil {
                log.Println(err)
                return
            }
            info, _ := os.Stat(thumb) // OK to ignore error
            sizes <- info.Size()
        }(f)
    }

    // closer
    go func() {
        wg.Wait()
        close(sizes)
    }()


    var total int64
    for size := range sizes {
        total += size
    }
    return total
}

观察一下我们是怎样创建一个closer goroutine，并让其等待worker们在关闭掉sizes channel之前退出的。两步操作：wait和close，必须是基于sizes的循环的并发。考虑一下另一种方案：

如果等待操作被放在了main goroutine中，在循环之前，这样的话就永远都不会结束了，
//wait()阻塞main goroutine, 循环执行不了


如果在循环之后，那么又变成了不可达的部分，因为没有任何东西去关闭这个channel，这个循环就永远都不会终止。
//没人关闭sizes 后面一直接受0无限循环


make和new

new 的作用是 初始化 一个指向类型的指针 (*T)， make 的作用是为 slice, map 或者 channel 初始化，并且返回引用 T
make(T, args)函数的目的与new(T)不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是T*）的一个初始化的（不是零值）的实例。 这中差别的出现是由于这三种类型实质上是对在使用前必须进行初始化的数据结构的引用。 例如, Slice是一个 具有三项内容的描述符，包括 指向数据（在一个数组内部）的指针，长度以及容量。在这三项内容被初始化之前，Slice的值为nil。对于Slice，Map和Channel， make（）函数初始化了其内部的数据结构，并且准备了将要使用的值。

- 切片、映射和通道，使用make   Slice, Map 和 Channel
- 数组、结构体和所有的值类型，使用new 


nil
interface 
func checkError(err error) {
    if err != nil {
        panic(err)
    }
}
var e *Error
checkError(e)

e是nil的，但是当我们checkError时就会panic。请读者思考一下为什么？


string和slice
string的空值是""，它是不能跟nil比较的。即使是空的string，它的大小也是两个机器字长的。slice也类似，它的空值并不是一个空指针，而是结构体中的指针域为空，空的slice的大小也是三个机器字长的。


channel和map
channel跟string或slice有些不同，它在栈上只是一个指针，实际的数据都是由指针所指向的堆上面。
跟channel相关的操作有：初始化/读/写/关闭。channel未初始化值就是nil，未初始化的channel是不能使用的。下面是一些操作规则：
  ● 读或者写一个nil的channel的操作会永远阻塞。
  ● 读一个关闭的channel会立刻返回一个channel元素类型的零值。
  ● 写一个关闭的channel会导致panic。
map也是指针，实际数据在堆中，未初始化的值是nil。


defer

return xxx这一条语句并不是一条原子指令!
返回值 = xxx
调用defer函数
空的return


func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}
1

func f() (r int) {
     t := 5
     defer func() {
       t = t + 5
     }()
     return t
}

5

func f() (r int) {
    defer func(r int) {
          r = r + 5
    }(r)
    return 1
}

1




for _, file := range files {
    if f, err = os.Open(file); err != nil {
        return
    }
    // 这是错误的方式，当循环结束时文件没有关闭
    defer f.Close()
    // 对文件进行操作
    f.Process(data)
}
defer仅在函数返回时才会执行，在循环的结尾或其他一些有限范围的代码内不会执行。

for _, file := range files {
    if f, err = os.Open(file); err != nil {
        return
    }
    // 对文件进行操作
    f.Process(data)
    // 关闭文件
    f.Close()
 }





误用短声明导致变量覆盖

func shadow() (err error) {
	x, err := check1() // x是新创建变量，err是被赋值
	if err != nil {
		return // 正确返回err
	}
	if y, err := check2(x); err != nil { // y和if语句中err被创建
		return // if语句中的err覆盖外面的err，所以错误的返回nil！
	} else {
		fmt.Println(y)
	}
	return
}

不需要将一个指向切片的指针传递给函数
切片实际是一个指向潜在数组的指针。我们常常需要把切片作为一个参数传递给函数是因为：实际就是传递一个指向变量的指针，在函数内可以改变这个变量，而不是传递数据的拷贝。
因此应该这样做：
    `func findBiggest( listOfNumbers []int ) int {}`
而不是：
   `func findBiggest( listOfNumbers *[]int ) int {}` 
当切片作为参数传递时，切记不要解引用切片。



不要使用一个指针指向一个接口类型，因为它已经是一个指针。

func nextFew1(n nexter, num int) []byte {
    var b []byte
    for i:=0; i < num; i++ {
        b[i] = n.next()
    }
    return b
}
func nextFew2(n *nexter, num int) []byte {
    var b []byte
    for i:=0; i < num; i++ {
        b[i] = n.next() // 编译错误:n.next未定义（*nexter类型没有next成员或next方法）
    }
    return b
}
func main() {
    fmt.Println(“Hello World!”)
}



err defer return

"An error is returned if caused by client policy (such as CheckRedirect), or if there was an HTTP protocol error. A non-2xx response doesn't cause an error.
When err is nil, resp always contains a non-nil resp.Body."
Then looking at this code:
res, err := client.Do(req)
defer res.Body.Close()

if err != nil {
    return nil, err
}
I'm guessing that err is not nil. You're accessing the .Close() method on res.Body before you check for the err.
The defer only defers the function call. The field and method are accessed immediately.
--------------------------------------------------------------------------------
So instead, try checking the error immediately.
res, err := client.Do(req)

if err != nil {
    return nil, err
}
defer res.Body.Close()

