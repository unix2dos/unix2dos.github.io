---
title: nginx全局变量
tags:
  - nginx
abbrlink: 80b24c5f
categories:
  - linux系统运维
  - nginx
date: 2019-07-06 11:44:46
---

### 1. 服务器相关

| 变量名                | 备注                                                         | 示例                                               |
| --------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| `nginx_version`       | 当前运行的 Nginx 版本号                                      | 1.11.2                                             |
| `server_port`         | 服务器端口                                                   | 8080                                               |
| `server_addr`         | 服务器端地址                                                 | 127.0.0.1                                          |
| `server_name`         | 服务器名称                                                   | 127.0.0.1                                          |
| `server_protocol`     | 服务器的HTTP版本                                             | HTTP/1.0                                           |
| `status`              | HTTP 响应代码                                                | 200                                                |
| `time_iso8601`        | 服务器时间的 ISO 8610 格式                                   | 2018-09-02T15:14:27+08:00                          |
| `time_local`          | 服务器时间（LOG Format 格式）                                | 02/Sep/2018:15:14:27 +0800                         |
| `document_root`       | 当前请求的文档根目录或别名                                   | `/home/xiaowu/github/echo.xuexb.com`               |
| `request_filename`    | 当前连接请求的文件路径，由 `root`或 `alias`指令与 URI 请求生成 | `/home/xiaowu/github/echo.xuexb.com/api/dump/path` |
| `request_completion`  | 如果请求成功，值为”OK”，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空 |                                                    |
| `pid`                 | 工作进程的PID                                                | 1234                                               |
| `msec`                | 当前的Unix时间戳                                             | 1535872750.954                                     |
| `limit_rate`          | 用于设置响应的速度限制                                       | 0                                                  |
| `pipe`                | 如果请求来自管道通信，值为“p”，否则为“.”                     | .                                                  |
| `connection_requests` | TCP连接当前的请求数量                                        | 1                                                  |
| `connection`          | TCP 连接的序列号                                             | 363861                                             |
| `realpath_root`       | 当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径 | /home/xiaowu/github/echo.xuexb.com                 |
|                       

<!-- more -->

### 2. 客户端相关

| 变量名              | 示例                                                         | 备注                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| host                | echo.xuexb.com                                               | 优先级如下：HTTP请求行的主机名>”HOST”请求头字段>符合请求的服务器名 |
| hostname            | bj01                                                         | 主机名                                                       |
| remote_port         | 58500                                                        | 客户端端口                                                   |
| remote_user         |                                                              | 用于HTTP基础认证服务的用户名                                 |
| request             | GET /api/dump/path?a=1&%E4%B8%AD%E6%96%87=%E5%A5%BD%E7%9A%84%23123 HTTP/1.0 | 代表客户端的请求地址                                         |
| remote_addr         | 127.0.0.1                                                    | 客户端地址                                                   |
| request_body        |                                                              | 客户端的请求主体, 此变量可在location中使用，将请求主体通过proxy_pass, fastcgi_pass, uwsgi_pass, 和 scgi_pass传递给下一级的代理服务器 |
| request_body_file   |                                                              | 将客户端请求主体保存在临时文件中文件处理结束后，此文件需删除如果需要之一开启此功能，需要设置client_body_in_file_only如果将次文件传递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off, uwsgi_pass_request_body off, or scgi_pass_request_body off |
| proxy_protocol_addr |                                                              | 获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串(1.5.12) |
| http_名称           | http_accept_language -> zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7  | 匹配任意请求头字段； 变量名中的后半部分“name”可以替换成任意请求头字段，如在配置文件中需要获取http请求头：“Accept-Language”，那么将“－”替换为下划线，大写字母替换为小写，形如：http_accept_language即可 |
| bytes_sent          | 0                                                            | 传输给客户端的字节数 (1.3.8, 1.2.5)                          |
| body_bytes_sent     | 0                                                            | 传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的“%B”参数保持兼容 |



### 3. 客户端相关 - request headers

| 变量名                           | 备注                                                         | 示例                                                         |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `http_accept`                    | 浏览器支持的 MIME 类型                                       | `text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8` |
| `http_accept_encoding`           | 浏览器支持的压缩编码                                         | `gzip, deflate, br`                                          |
| `http_accept_language`           | 浏览器支持的语言                                             | `zh-CN,zh;q=0.9,en;q=0.8`                                    |
| `http_cache_control`             | 浏览器缓存                                                   | `max-age=0`                                                  |
| `http_connection`                | 客户端与服务连接类型                                         |                                                              |
| `http_cookie`                    | 浏览器请求 cookie                                            | `a=1; b=2`                                                   |
| `http_host`                      | 浏览器请求 host                                              | echo.xuexb.com                                               |
| `http_referer`                   | 浏览器来源                                                   | https://echo.xuexb.com/                                      |
| `http_upgrade_insecure_requests` | 是一个请求首部，用来向服务器端发送信号，表示客户端优先选择加密及带有身份验证的响应，并且它可以成功处理 upgrade-insecure-requests CSP 指令 | 1                                                            |
| `http_user_agent`                | 用户设备标识                                                 | `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36` |
| `http_x_requested_with`          | 异步请求标识                                                 | true                                                         |
| `http_x_forwarded_for`           | 反向代理原 IP                                                | 198.13.61.105                                                |



### 4. 链接相关

| 变量名           | 备注                                                         | 示例                                                       |
| ---------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| `scheme`         | 请求使用的 WEB 协议                                          | http                                                       |
| `uri`            | 请求中的当前 URI(不带请求参数)，可以不同于浏览器传递的 `$request_uri` 的值，它可以通过内部重定向，或者使用 `index` 指令进行修改 | `/api/dump/path`                                           |
| `document_uri`   | 同 `$uri`                                                    | `/api/dump/path`                                           |
| `request_uri`    | 这个变量等于包含一些客户端请求参数的原始 URI ，它无法修改    | `/api/dump/path?a=1&%E4%B8%AD%E6%96%87=%E5%A5%BD%E7%9A%84` |
| `request_method` | HTTP 请求方法                                                | GET                                                        |
| `request_time`   | 处理客户端请求使用的时间，从读取客户端的第一个字节开始计时   | 0.000                                                      |
| `request_length` | 请求的长度（包括请求地址、请求头和请求主体）                 | 678                                                        |
| `args`           | 请求参数                                                     | `a=1&%E4%B8%AD%E6%96%87=%E5%A5%BD%E7%9A%84`                |
| `query_string`   | 同 `$args`                                                   |                                                            |
| `is_args`        | 请求中是否有参数，有则为 `?` 否则为空                        | `?`                                                        |
| `arg_参数名`     | 请求中具体的参数                                             | `$arg_a` => `1`                                            |
| `https`          | 如果开启了 SSL 安全模式，则为 `on` 否则为空                  | `on`                                                       |



### 5. 参考资料

+ https://xuexb.github.io/learn-nginx/variable/
+ https://nginx.org/en/docs/



