---
title: "记录一次docker镜像的构建过程"
date: 2019-06-27 20:54:46
tags:
- docker
- linux
---



Docker 官方提供了一些建议和准则，在大多数情况下建议遵守。

1、容器是短暂的，也就是说，你需要可以容易的创建、销毁、配置你的容器。

2、多数情况，构建镜像的时候是将 Dockerfile 和所需文件放在同一文件夹下。但为了构建性能，我们可以采用 [.dockerignore](https://deepzz.com/post/dockerfile-reference.html#toc_6) 文件来排除文件和目录。

3、避免安装不必要的包，构建镜像应该尽可能减少复杂性、依赖关系、构建时间及镜像大小。

4、最小化层数。 Dockerfile的一行(除MAINTAINER外)对应镜像的一层，为使层数足够小，故可以将类似的命令串起来，比如RUN 指令，可以使用&&连接多个指令，如此也只有一层。

5、排序多行参数，通过字母将参数排序来缓解以后的变化，这将帮你避免重复的包、使列表更容易更新，如：

```dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

<!-- more -->



###  1. FROM 

尽可能的使用官方仓库存储的镜像作为基础镜像。官方建议使用 [Debian](https://hub.docker.com/_/debian/)，大小在 150mb 左右。不过在实际开发中，应该用到 [alpine](https://hub.docker.com/_/alpine/) 的次数比较多，因为它仅 5mb 左右。 busybox更只有1M多。



+ 参考: https://blog.csdn.net/bbwangj/article/details/81088231

  

### 2. TODO: 其他指令

+ copy 和 add的区别
+ run 和 cmd的区别

+ ENTRYPOINT 和 CMD的区别

+ CMD 容器内没有后台服务的概念。容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

+ CMD 运行可执行程序, 阻塞才可以, 要不然会退出

+ docker run 镜像 cmd.    在镜像名后面的是 command，运行时会替换 CMD 的默认值。

+ ENTRYPOINT 就是为了传递参数, 这当存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT。

+ ENTRYPOINT 执行一个脚本, 在cmd前做一些准备工作后, 执行cmd

+ 环境变量写法

```dockerfile
ENV KS_HAVEN_ADDR=':16097' \
KS_HAVEN_QUIC_ADDR=':16097'
```



### 3. 构建镜像

在 Dockerfile 文件所在目录执行：    `docker build -t image_name .`



##### 3.1 构建的上下文

1. c/s架构, 在服务端构建

2. 服务端要获取文件, 需要把上下文目录打包发过去 (COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法成功, 超出了上下文)

3. 最好将dockerfile放在空目录下或项目根目录下, 如果没有所需文件,拷贝过来, 避免发送太多文件给引擎



##### 3.2 运行容器

```bash
docker images #查看构建的image

docker run -d -p 16097:16097 -p 15098:15098  image_name # 启动容器

docker extc -it 容器名字 bash # 进入容器内部
docker exec -it 容器ID  sh   # 进入容器内部
```



##### 3.3 镜像save load

```bash
docker save fhyx/haven > haven.tar.gz # save image

docker load -i haven.tar.gz # load image
```



### 4. 容器操作



##### 4.1 看容器log

```bash
docker logs -f CONTAINER_ID

docker logs -f --tail=100 CONTAINER_ID
```



##### 4.2 容器拷贝内容

```bash
docker cp foo.txt mycontainer:/foo.txt 
docker cp mycontainer:/foo.txt foo.txt
```



##### 4.3 容器访问宿主主机端口

+ https://jingsam.github.io/2018/10/16/host-in-docker.html



### 5. 参考资料

+ https://blog.csdn.net/wdq347/article/details/78753322 docker之镜像制作
+ https://deepzz.com/post/dockerfile-best-practices.html  如何写好Dockerfile，Dockerfile最佳实践