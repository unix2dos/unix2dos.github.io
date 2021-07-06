---
title: openvpn搭建虚拟局域网
tags:
  - linux
  - openvpn
categories:
  - 2-linux系统
  - 软件
abbrlink: a84d9911
date: 2019-11-23 16:11:40
---



### 0. 前言

OpenVPN 是一个健壮的、高度灵活的 [VPN](https://en.wikipedia.org/wiki/VPN) 守护进程。它支持 [SSL/TLS](https://en.wikipedia.org/wiki/SSL/TLS) 安全、[Ethernet bridging](https://en.wikipedia.org/wiki/Bridging_(networking))、经由[代理](https://en.wikipedia.org/wiki/Proxy_server)的 [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) 或 [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) [隧道](https://en.wikipedia.org/wiki/Tunneling_protocol)和 [NAT](https://en.wikipedia.org/wiki/Network_address_translation)。另外，它也支持动态 IP 地址以及 [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)，可伸缩性足以支持数百或数千用户的使用场景，同时可移植至大多数主流操作系统平台上。

##### 安装openvpn

```bash
sudo apt install openvpn
```

<!-- more -->



### 1. 生成证书

```bash
git clone https://github.com/OpenVPN/easy-rsa
cd easyrsa3
```



##### 1.1 生成 CA

```bash
./easyrsa init-pki

./easyrsa build-ca  
# 输入密码: 123456
Common Name: OpenVPN-CA
```

pki文件夹下会生成 ca.crt



##### 1.2 生成server和 client 公钥私钥对

```bash
./easyrsa build-server-full server

./easyrsa build-client-full client1
./easyrsa build-client-full client2
./easyrsa build-client-full client3
```

pki/private 是私有的 key

pki/issued 是公有的 key



##### 1.3 生成Diffie-Hellman pem 

```bash
./easyrsa gen-dh
```

pki 文件夹下生成了 dh.pem



##### 1.4 现在我们有了

| **Filename** | **Needed By**            | **Purpose**               | **Secret** |
| ------------ | ------------------------ | ------------------------- | ---------- |
| ca.crt       | server + all clients     | Root CA certificate       | NO         |
| ca.key       | key signing machine only | Root CA key               | YES        |
| dh{n}.pem    | server only              | Diffie Hellman parameters | NO         |
| server.crt   | server only              | Server Certificate        | NO         |
| server.key   | server only              | Server Key                | YES        |
| client1.crt  | client1 only             | Client1 Certificate       | NO         |
| client1.key  | client1 only             | Client1 Key               | YES        |
| client2.crt  | client2 only             | Client2 Certificate       | NO         |
| client2.key  | client2 only             | Client2 Key               | YES        |
| client3.crt  | client3 only             | Client3 Certificate       | NO         |
| client3.key  | client3 only             | Client3 Key               | YES        |



### 2. 配置文件

在安装目录下(`/usr/share/doc/openvpn/examples/sample-config-files`)找到配置 `server.conf` and `client.conf`

如果只有有 server.conf.gz 的话, 需要解压

```bash
gunzip -c server.conf.gz > server.conf
```



##### 2.1  服务端修改证书路径

Before you use the sample configuration file, you should first edit the **ca**, **cert**, **key**, and **dh** parameters to point to the files you generated in the [PKI](https://openvpn.net/community-resources/how-to/#setting-up-your-own-certificate-authority-ca-and-generating-certificates-and-keys-for-an-openvpn-server-and-multiple-clients) section above.

##### 2.2 服务端dev 模式可以修改  tap tun

##### 2.3 服务端修改 ip 范围

If you want to use a virtual IP address range other than `10.8.0.0/24`, you should modify the `server` directive. Remember that this virtual IP address range should be a private range which is currently unused on your network.



The Internet Assigned Numbers Authority (IANA) has reserved the following three blocks of the IP address space for private internets (codified in RFC 1918):

| 10.0.0.0    | 10.255.255.255  | (10/8 prefix)       |
| ----------- | --------------- | ------------------- |
| 172.16.0.0  | 172.31.255.255  | (172.16/12 prefix)  |
| 192.168.0.0 | 192.168.255.255 | (192.168/16 prefix) |

The best candidates are subnets in the middle of the vast 10.0.0.0/8 netblock (for example 10.66.77.0/24).



##### 2.4 服务端修改 client 之间可连接

Uncomment out the `client-to-client` directive if you would like connecting clients to be able to reach each other over the VPN. By default, clients will only be able to reach the server.

##### 2.5 服务端修改 user 和 group

If you are using Linux, BSD, or a Unix-like OS, you can improve security by uncommenting out the **user nobody** and **group nobody** directives.

##### 2.6 客户端修改证书路径

`ca`, `cert`, `key`

##### 2.7 客户端修改 remote 参数

```bash
remote my-server-1 1194
```



##### 2.8 服务器和客户端的 `dev` (tun or tap) and `proto` (udp or tcp) 要一致



### 3. 启动使用

##### 3.1 启动 server

```bash
sudo openvpn /etc/openvpn/server.conf
```

成功启动以后会发现多了一个 tun 网口



如遇到错误: [Open VPN options error: --tls-auth fails with 'ta.key': no such file or directory](https://unix.stackexchange.com/questions/359428/open-vpn-options-error-tls-auth-fails-with-ta-key-no-such-file-or-director)

```bash
sudo openvpn --genkey --secret /etc/openvpn/certs/ta.key
```



##### 3.2 启动 client

``` bash
sudo openvpn /etc/openvpn/client.conf
```



如遇到错误: Authenticate/Decrypt packet error: packet HMAC authentication failed, 配置文件里,

```bash
tls-auth /etc/openvpn/certs/ta.key 0  #服务器用0

tls-auth /etc/openvpn/certs/ta.key 1  #客户端用1
```

注意, 这个 key 是同一个, 在服务器生成, 不是每个都生成一次



##### 3.3 测试

在客户端  `ping 10.8.0.1`

If the ping succeeds, congratulations! You now have a functioning VPN.

我们也可以 `ssh user@10.8.0.1` 发现也可以



##### 3.4 mac 使用

https://tunnelblick.net/ 下载安装包

可以生成.ovpn 文件, 参考https://serverfault.com/a/483967



### 4. openvpn服务

##### 4.1 server service

```bash
sudo systemctl start openvpn@server.service

sudo systemctl enable openvpn@server.service
```

需要输入密码请这样

```bash
sudo systemd-tty-ask-password-agent 
```



##### 4.2 client service

```bash
sudo systemctl start openvpn@client.service

sudo systemctl enable openvpn@client.service
```



### 5. 客户端分配固定 IP

```bash
cd /etc/openvpn
mkdir ccd

# 配置文件修改client-config-dir
vim server.conf
client-config-dir ccd

#在ccd文件夹下建立以用户名(Common Name)为名称的文件
cd ccd

vi client1
ifconfig-push 10.8.0.2 255.255.255.0

vi client2
ifconfig-push 10.8.0.3 255.255.255.0
```



### 6. 参考资料

+ https://openvpn.net/community-resources/how-to/

+ https://github.com/OpenVPN/easy-rsa

+ https://tunnelblick.net/

+ https://community.openvpn.net/openvpn/wiki/Concepts-Addressing