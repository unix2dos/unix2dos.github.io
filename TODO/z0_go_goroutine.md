#### 数据竞争

1. 提前初始化好变量
2. 数据在一个gorouting里修改读取(不能一个解决的串行gorouting)

```
    go func(){
        for {
            select{
            }
        }
    }()
```
3. 加锁


#### 内存同步


##### 在一个独立的goroutine中，每一个语句的执行顺序是可以被保证的，也就是说goroutine内顺序是连贯的。

但是在不使用channel且不使用mutex这样的显式同步操作时，我们就没法保证事件在不同的goroutine中看到的执行顺序是一致的了。


#### 缓存无锁
1. 为了效率加上缓存, 但是get的时候存在竞争
    
```
func httpGetBody(url string) (interface{}, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	return ioutil.ReadAll(resp.Body)
}

type Func func(key string) (interface{}, error)

type result struct {
	value interface{}
	err   error
}

type entry struct {
	res   result
	ready chan struct{} // closed when res is ready
}

func New(f Func) *Memo {
	return &Memo{f: f, cache: make(map[string]*entry)}
}

type Memo struct {
	f     Func
	mu    sync.Mutex // guards cache
	cache map[string]*entry
}

func (memo *Memo) Get(key string) (value interface{}, err error) {
	memo.mu.Lock()
	e := memo.cache[key]
	if e == nil {
		e = &entry{ready: make(chan struct{})}
		memo.cache[key] = e
		memo.mu.Unlock()
		e.res.value, e.res.err = memo.f(key)
		close(e.ready)
	} else {
		memo.mu.Unlock()
		<-e.ready
	}
	return e.res.value, e.res.err
}

func main() {
	memo := New(httpGetBody)
	go func() {
		value, err := memo.Get("https://www.baidu.com")
		fmt.Println(string(value.([]byte)), err)
	}()
	go func() {
		value, err := memo.Get("https://www.baidu.com")
		fmt.Println(string(value.([]byte)), err)
	}()

	time.Sleep(time.Second * 5)
}

```

#### 完全不用锁

```

func httpGetBody(url string) (interface{}, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	return ioutil.ReadAll(resp.Body)
}

type Func func(key string) (interface{}, error)

//request
type request struct {
	key      string
	response chan<- result
}

//reult
type result struct {
	value interface{}
	err   error
}

//reultC
type entry struct {
	res   result
	ready chan struct{}
}

//Memo类
type Memo struct {
	requests chan request
}

func (m *Memo) testChannel(f Func) {
	cache := make(map[string]*entry)
	for req := range m.requests {
		e := cache[req.key]
		if e == nil {
			e = &entry{ready: make(chan struct{})}
			cache[req.key] = e

			go func(e *entry, req request) {
				e.res.value, e.res.err = f(req.key)
				close(e.ready)
			}(e, req)//此处参数一定要传递进去, for循环引用问题
		}

		go func(e *entry, req request) {
			<-e.ready
			req.response <- e.res
		}(e, req)//此处参数一定要传递进去, for循环引用问题
	}

}

func (memo *Memo) Get(key string) (value interface{}, err error) {
	response := make(chan result)
	memo.requests <- request{key, response}
	res := <-response
	return res.value, res.err
}

func (m *Memo) Close() {
	close(m.requests)
}

func New(f Func) *Memo {
	memo := &Memo{requests: make(chan request)}
	go memo.testChannel(f)
	return memo
}

func main() {
	memo := New(httpGetBody)
	go func() {
		value, err := memo.Get("https://www.baidu.com")
		fmt.Println("11111111")
		fmt.Println(string(value.([]byte)), err)
	}()
	go func() {
		value, err := memo.Get("https://www.baidu.com")
		fmt.Println("222222222222")
		fmt.Println(string(value.([]byte)), err)
	}()

	time.Sleep(time.Second * 5)
}
```

#### goroutine 调度

```
for {
	go fmt.Print(0)
	fmt.Print(1)
}

$ GOMAXPROCS=1 go run hacker-cliché.go
111111111111111111110000000000000000000011111...

$ GOMAXPROCS=2 go run hacker-cliché.go
010101010101010101011001100101011010010100110...
```

