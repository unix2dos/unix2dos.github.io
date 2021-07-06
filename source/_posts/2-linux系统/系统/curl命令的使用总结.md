---
title: curl命令的使用总结
tags:
  - curl
  - linux
categories:
  - 2-linux系统
  - 系统
abbrlink: da64728
date: 2019-10-19 11:11:46
---

[curl](http://curl.haxx.se/)是一种命令行工具，作用是发出网络请求，然后得到和提取数据，显示在"标准输出"（stdout）上面。

### 1. 使用教程

##### 1.1 查看网页源码和保存

```bash
curl www.sina.com
```

如果要把这个网页保存下来，可以使用`-o`参数，这就相当于使用wget命令了。

```bash
curl -o [文件名] www.sina.com
```

<!-- more -->

##### 1.2 显示响应头信息

`-i`参数可以显示http response的头信息，连同网页代码一起。`-I`参数则是只显示http response的头信息。

```bash
curl -i www.sina.com
```



##### 1.3 显示通信过程

`-v`参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。

```bash
curl -v www.sina.com
```

如果你觉得上面的信息还不够，那么下面的命令可以查看更详细的通信过程。

```bash
curl --trace output.txt www.sina.com
curl --trace-ascii output.txt www.sina.com
```

运行后，请打开output.txt文件查看。



### 2. 发送数据

##### 2.1 发送GET

GET方法相对简单，只要把数据附在网址后面就行。

```bash
curl example.com/form.cgi?data=xxx
```



##### 2.2 发送POST

POST方法必须把数据和网址分开，curl就要用到--data参数。

```bash
curl -X POST --data "data=xxx" example.com/form.cgi
```

如果你的数据没有经过表单编码，还可以让curl为你编码，参数是`--data-urlencode`。

```bash
curl -X POST --data-urlencode "date=April 1" example.com/form.cgi

# 如果编码前有=号, 需要提前编码
--data-urlencode  'value=-vf scale=-2:360'   # 错误
--data-urlencode  'value=-vf scale%3d-2:360' # 正确
```



##### 2.3 HTTP动词

curl默认的HTTP动词是GET，使用`-X`参数可以支持其他动词。

```bash
curl -X DELETE www.example.com
```



##### 2.4 增加头信息

有时需要在http request之中，自行增加一个头信息。`--header` 或 `-H `参数就可以起到这个作用。

```bash
curl --header "Content-Type:application/json" http://example.com
```



##### 2.5 HTTP认证

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```bash
curl --user name:password example.com
```



##### 2.6 cookie

使用`--cookie`参数，可以让curl发送cookie。

```bash
curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从http response头信息的`Set-Cookie`字段中得到。



`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```bash
curl -c cookies http://example.com
curl -b cookies http://example.com
```



##### 2.7 User Agent字段

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

```bash
curl --user-agent "[User Agent]" [URL]
```



##### 2.8 Referer字段

有时你需要在http request头信息中，提供一个referer字段，表示你是从哪里跳转过来的。

```bash
curl --referer http://www.example.com http://www.example.com
```



##### 2.9 文件上传

假定文件上传的表单是下面这样：

```html
<form method="POST" enctype='multipart/form-data' action="upload.cgi">
　　　　<input type=file name=upload>
　　　　<input type=submit name=press value="OK">
　　</form>
```

你可以用curl这样上传文件：

```bash
curl --form upload=@localfilename --form press=OK [URL]
```



##### 2.10 终极命令

```bash
curl --help
```

