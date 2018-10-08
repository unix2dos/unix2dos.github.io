1. 自动生成test https://github.com/cweill/gotests
2. test 验证  https://github.com/stretchr/testify




+ 生成随机bytes

```
func randomBytes(n int) []byte {
	b := make([]byte, n)
	rand.Read(b)
	return b
}
```


+ 获取外网IP

```
func getRemoteIP() string {
	client := &http.Client{Timeout: time.Second * 30}
	req, _ := http.NewRequest("GET", "http://ifconfig.me", nil)
	req.Header.Set("User-Agent", "curl/7.29.0")

	res, _ := client.Do(req)
	defer res.Body.Close()

	bytes, _ := ioutil.ReadAll(res.Body)
	return string(bytes)

}
```

+ 获取本地ips

```

func getLocalIPS() (ips []string) {

	ips = make([]string, 0, 6)

	addrs, err := net.InterfaceAddrs()
	if err != nil {
		return
	}

	for _, arr := range addrs {
		ip, _, err := net.ParseCIDR(arr.String())
		if err != nil {
			continue
		}
		ips = append(ips, ip.String())
	}

	return
}
```

+ 线程安全map

```
type Item struct {
	key interface{}
	val interface{}
}

type SyncMap struct {
	*sync.RWMutex
	data map[interface{}]interface{}
}

func NewSyncMap() *SyncMap {
	return &SyncMap{
		RWMutex: &sync.RWMutex{},
		data:    make(map[interface{}]interface{}),
	}
}

func (m *SyncMap) Set(key interface{}, val interface{}) {
	m.Lock()
	defer m.Unlock()
	m.data[key] = val
}

func (m *SyncMap) Get(key interface{}) (interface{}, bool) {
	m.RLock()
	defer m.RUnlock()
	val, ok := m.data[key]
	return val, ok
}

func (m *SyncMap) Delete(key interface{}) {
	m.Lock()
	defer m.Unlock()
	delete(m.data, key)
}

func (m *SyncMap) DeleteMulti(keys []interface{}) {
	m.Lock()
	defer m.Unlock()
	for k, _ := range keys {
		delete(m.data, k)
	}
}

func (m *SyncMap) Clear() {
	m.Lock()
	defer m.Unlock()
	m.data = make(map[interface{}]interface{})
}

func (m *SyncMap) Len() int {
	m.RLock()
	defer m.RUnlock()
	return len(m.data)
}

func (m *SyncMap) Iter() <-chan Item {
	ch := make(chan Item)

	go func() {
		m.RLock()
		for k, v := range m.data {
			ch <- Item{k, v}
		}
		m.RUnlock()

		close(ch)
	}()

	return ch
}

func main() {
	map1 := NewSyncMap()
	map1.Set("1", "liuwei")
	map1.Set("2", "xuanyuan")

	for ch := range map1.Iter() {
		fmt.Println(ch.key)
		fmt.Println(ch.val)
	}
}
```







