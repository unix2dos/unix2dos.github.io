####  socket

在Go的net包中定义了很多类型、函数和方法用来网络编程，其中IP的定义如下：

type IP []byte
其中ParseIP(s string) IP函数会把一个IPv4或者IPv6的地址转化成IP类型



#### TCP Socket

在Go语言的net包中有一个类型TCPConn，这个类型可以用来作为客户端和服务器端交互的通道，他有两个主要的函数：

```
func (c *TCPConn) Write(b []byte) (int, error)
func (c *TCPConn) Read(b []byte) (int, error)
```

还有我们需要知道一个TCPAddr类型，他表示一个TCP的地址信息，他的定义如下：

```
type TCPAddr struct {
	IP IP
	Port int
	Zone string // IPv6 scoped addressing zone
}
```

在Go语言中通过ResolveTCPAddr获取一个TCPAddr

```
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```
net参数是"tcp4"、"tcp6"、"tcp"中的任意一个，分别表示TCP(IPv4-only), TCP(IPv6-only)或者TCP(IPv4, IPv6的任意一个)。
addr表示域名或者IP地址，例如"www.google.com:80" 或者"127.0.0.1:22"。

#### client

```
func DialTCP(network string, laddr, raddr *TCPAddr) (*TCPConn, error)
```

net参数是"tcp4"、"tcp6"、"tcp"中的任意一个，分别表示TCP(IPv4-only)、TCP(IPv6-only)或者TCP(IPv4,IPv6的任意一个)
laddr表示本机地址，一般设置为nil
raddr表示远程的服务地址


#### server 

```
func ListenTCP(network string, laddr *TCPAddr) (*TCPListener, error)
func (l *TCPListener) Accept() (Conn, error)
```

### control

```
func DialTimeout(net, addr string, timeout time.Duration) (Conn, error)
```
设置建立连接的超时时间，客户端和服务器端都适用，当超过设置时间时，连接自动关闭。




```
func (c *TCPConn) SetReadDeadline(t time.Time) error
func (c *TCPConn) SetWriteDeadline(t time.Time) error
```
用来设置写入/读取一个连接的超时时间。当超过设置时间时，连接自动关闭。



```
func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error
```
设置keepAlive属性，是操作系统层在tcp上没有数据和ACK的时候，会间隔性的发送keepalive包，操作系统可以通过该包来判断一个tcp连接是否已经断开，在windows上默认2个小时没有收到数据和keepalive包的时候人为tcp连接已经断开，这个功能和我们通常在应用层加的心跳包的功能类似。

### udp functions

```
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error)
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)
```

