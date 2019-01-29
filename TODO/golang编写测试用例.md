Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试，`testing`框架和其他语言中的测试框架类似，你可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例，那么接下来让我们一一来看一下怎么写。

另外建议安装[gotests](https://github.com/cweill/gotests)插件自动生成测试代码:

```shell
go get -u -v github.com/cweill/gotests/...
```



##  1. testing - 单元测试

`testing` 为 Go 语言 package 提供自动化测试的支持。通过 `go test` 命令，能够自动执行如下形式的任何函数：

```
func TestXxx(*testing.T)
```

注意：Xxx 可以是任何字母数字字符串，但是第一个字母不能是小些字母。

在这些函数中，使用 Error, Fail 或相关方法来发出失败信号。

要编写一个新的测试套件，需要创建一个名称以 _test.go 结尾的文件，该文件包含 `TestXxx` 函数，如上所述。 将该文件放在与被测试的包相同的包中。该文件将被排除在正常的程序包之外，但在运行 “go test” 命令时将被包含。 有关详细信息，请运行 “go help test” 和 “go help testflag” 了解。

如果有需要，可以调用 `*T` 和 `*B` 的 Skip 方法，跳过该测试或基准测试：

```go
func TestTimeConsuming(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping test in short mode.")
    }
    ...
}
```



### 1. 1 Table-Driven Test

测试讲究 case 覆盖，按上面的方式，当我们要覆盖更多 case 时，显然通过修改代码的方式很笨拙。这时我们可以采用 Table-Driven 的方式写测试，标准库中有很多测试是使用这种方式写的。

```go
func TestFib(t *testing.T) {
    var fibTests = []struct {
        in       int // input
        expected int // expected result
    }{
        {1, 1},
        {2, 1},
        {3, 2},
        {4, 3},
        {5, 5},
        {6, 8},
        {7, 13},
    }

    for _, tt := range fibTests {
        actual := Fib(tt.in)
        if actual != tt.expected {
            t.Errorf("Fib(%d) = %d; expected %d", tt.in, actual, tt.expected)
        }
    }
}
```

因为我们使用的是 `t.Errorf`，其中某个 case 失败，并不会终止测试执行。

### 1. 2 T 类型

单元测试中，传递给测试函数的参数是 `*testing.T` 类型。它用于管理测试状态并支持格式化测试日志。测试日志会在执行测试的过程中不断累积，并在测试完成时转储至标准输出。

当一个测试的测试函数返回时，又或者当一个测试函数调用 `FailNow`、 `Fatal`、`Fatalf`、`SkipNow`、`Skip` 或者 `Skipf` 中的任意一个时，该测试即宣告结束。跟 `Parallel` 方法一样，以上提到的这些方法只能在运行测试函数的 goroutine 中调用。

至于其他报告方法，比如 `Log` 以及 `Error` 的变种， 则可以在多个 goroutine 中同时进行调用。

### 1. 3 Parallel 测试

包中的 Parallel 方法用于表示当前测试只会与其他带有 Parallel 方法的测试并行进行测试。

下面的例子演示的 Parallel 的使用：

```go
var (
    data   = make(map[string]string)
    locker sync.RWMutex
)

func WriteToMap(k, v string) {
    locker.Lock()
    defer locker.Unlock()
    data[k] = v
}

func ReadFromMap(k string) string {
    locker.RLock()
    defer locker.RUnlock()
    return data[k]
}
```

测试代码：

```go
var pairs = []struct {
    k string
    v string
}{
    {"polaris", "徐新华"},
    {"studygolang", "Go语言中文网"},
    {"stdlib", "Go语言标准库"},
    {"polaris1", "徐新华1"},
    {"studygolang1", "Go语言中文网1"},
    {"stdlib1", "Go语言标准库1"},
    {"polaris2", "徐新华2"},
}

// 注意 TestWriteToMap 需要在 TestReadFromMap 之前
func TestWriteToMap(t *testing.T) {
    t.Parallel()
    for _, tt := range pairs {
        WriteToMap(tt.k, tt.v)
    }
}

func TestReadFromMap(t *testing.T) {
    t.Parallel()
    for _, tt := range pairs {
        actual := ReadFromMap(tt.k)
        if actual != tt.v {
            t.Errorf("the value of key(%s) is %s, expected: %s", tt.k, actual, tt.v)
        }
    }
}
```

试验步骤：

1. 注释掉 WriteToMap 和 ReadFromMap 中 locker 保护的代码，同时注释掉测试代码中的 t.Parallel，执行测试，测试通过，即使加上 `-race`，测试依然通过；(因为不是同时进行)
2. 只注释掉 WriteToMap 和 ReadFromMap 中 locker 保护的代码，执行测试，测试失败（如果未失败，加上 `-race` 一定会失败）；

如果代码能够进行并行测试，在写测试时，尽量加上 `Parallel`，这样可以测试出一些可能的问题。



## 2. testing的测试用例形式

测试用例有四种形式： 

+ TestXxxx(t *testing.T) // 基本测试用例*

* BenchmarkXxxx(b* testing.B) // 压力测试的测试用例 
* Example_Xxx() // 测试控制台输出的例子
*  TestMain(m *testing.M) // 测试Main函数

给个Example的例子:（Example需要在最后用注释的方式确认控制台输出和预期是不是一致的）

```go
func Example_GetScore() {
    score := getScore(100, 100, 100, 2.1)
    fmt.Println(score)
    // Output:
    // 31.1
}
```



## 3. testing的变量

gotest的变量有这些：

- test.short : 一个快速测试的标记，在测试用例中可以使用testing.Short()来绕开一些测试
- test.outputdir : 输出目录
- test.coverprofile : 测试覆盖率参数，指定输出文件
- test.run : 指定正则来运行某个/某些测试用例 (测试单个函数)
- test.memprofile : 内存分析参数，指定输出文件
- test.memprofilerate : 内存分析参数，内存分析的抽样率
- test.cpuprofile : cpu分析输出参数，为空则不做cpu分析
- test.blockprofile : 阻塞事件的分析参数，指定输出文件
- test.blockprofilerate : 阻塞事件的分析参数，指定抽样频率
- test.timeout : 超时时间
- test.cpu : 指定cpu数量
- test.parallel : 指定运行测试用例的并行数



## 4. testing包内的结构

- B : 压力测试
- BenchmarkResult : 压力测试结果
- Cover : 代码覆盖率相关结构体
- CoverBlock : 代码覆盖率相关结构体
- InternalBenchmark : 内部使用的结构
- InternalExample : 内部使用的结构
- InternalTest : 内部使用的结构
- M : main测试使用的结构
- PB : Parallel benchmarks 并行测试使用结果
- T : 普通测试用例
- TB : 测试用例的接口



## 5. testing - 基准测试















   