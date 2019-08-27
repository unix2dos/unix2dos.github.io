---
title: 记录一次docker镜像的构建过程
tags:
  - docker
  - linux
categories:
  - 2-linux系统
  - docker
abbrlink: 2450240c
date: 2019-08-24 20:54:46
---

在制作 Docker Images 之前, 我们先看一下Docker 官方提供了一些建议和准则，在大多数情况下建议遵守。

+ 容器是短暂的，也就是说，你需要可以容易的创建、销毁、配置你的容器。

+ 多数情况，构建镜像的时候是将 Dockerfile 和所需文件放在同一文件夹下。但为了构建性能，我们可以采用 [.dockerignore](https://deepzz.com/post/dockerfile-reference.html#toc_6) 文件来排除文件和目录。

+ 避免安装不必要的包，构建镜像应该尽可能减少复杂性、依赖关系、构建时间及镜像大小。

+ 最小化层数。 Dockerfile的一行(除MAINTAINER外)对应镜像的一层，为使层数足够小，故可以将类似的命令串起来，比如RUN 指令，可以使用&&连接多个指令，如此也只有一层。

+ 排序多行参数，通过字母将参数排序来缓解以后的变化，这将帮你避免重复的包、使列表更容易更新，如：

```dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

<!-- more -->



### 0. 前言

容器内没有后台服务的概念。容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。所以CMD 运行可执行程序, 阻塞才可以, 要不然会退出。



###  1. FROM 仓库

尽可能的使用官方仓库存储的镜像作为基础镜像。官方建议使用 [Debian](https://hub.docker.com/_/debian/)，大小在 150mb 左右。不过在实际开发中，应该用到 [alpine](https://hub.docker.com/_/alpine/) 的次数比较多，因为它仅 5mb 左右。 busybox更只有1M多。

参考: https://blog.csdn.net/bbwangj/article/details/81088231



### 2. 指令

##### 2.1 COPY 和 ADD 的区别

在大多数情况下使用COPY, 使用ADD的唯一原因就是你有一个压缩文件，你想自动解压到镜像中。

##### 2.2 RUN 和 CMD 的区别

+ RUN命令是创建Docker镜像的步骤，一个Dockerfile中可以有许多个RUN命令。
+ CMD命令是当Docker镜像被启动后Docker容器将会默认执行的命令。一个Dockerfile中只能有一个CMD命令。通过执行docker run $image other_command启动镜像可以重载CMD命令。

##### 2.3 ENTRYPOINT 和 CMD的区别

The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.

如果docker run没有指定任何的执行命令或者dockerfile里面也没有entrypoint，那么，就会使用cmd指定的默认的执行命令执行。同时也从侧面说明了entrypoint的含义，它才是真正的容器启动以后要执行命令。

+ CMD的用法

  ```
  The CMD instruction has three forms:
   
  CMD ["executable","param1","param2"] (exec form, this is the preferred form) //推荐
  CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
  CMD command param1 param2 (shell form)
  
  
  
  
  
  
  eg1: CMD ["/bin/bash", "-c", "echo 'hello cmd!'"]
  eg2: CMD ["hello cmd!"]
		 ENTRYPOINT ["echo"]
  eg3: CMD echo "hello cmd!"
  ```
  
  
  
+ entrypoint的用法

  An ENTRYPOINT allows you to configure a container that will run as an executable.

  ```
  ENTRYPOINT has two forms:
  
  ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred) //推荐
  ENTRYPOINT command param1 param2 (shell form)
  
  
  
  
  
  eg1: 如果命令后面有东西，那么后面的全部都会作为entrypoint的参数。如果没有，但是cmd有，那么cmd的全部内容会作为entrypoint的参数, 会输出 hello cmd 的
  
  CMD ["hello cmd!"]
  ENTRYPOINT ["echo"]
  
  
  
  
  eg2: 这个时候是不会输出 hello cmd 的
  
  CMD ["hello cmd!"]
  ENTRYPOINT echo
  ```

+ 覆盖问题

  + cmd 除非默认, 否则轻易被覆盖

  + entrypoint 可以用 --entrypoint 覆盖

  + 所以建议entrypoint固定, cmd 被覆盖, 结合使用, 并且永远使用Exec表示法



##### 2.9 环境变量写法

```
ENV KS_HAVEN_ADDR=':16097' \
KS_HAVEN_QUIC_ADDR=':16097'
```



### 3. 构建镜像

在 Dockerfile 文件所在目录执行：    `docker build -t image_name .`



##### 3.1 构建的上下文

1. c/s架构, 在服务端构建
2. 服务端要获取文件, 需要把上下文目录打包发过去 (COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法成功, 超出了上下文)
3. 最好将dockerfile放在空目录下或项目根目录下, 如果没有所需文件,拷贝过来, 避免发送太多文件给引擎



##### 3.2 镜像 save load

```bash
docker images #查看构建的image

docker save image/test > image_test.tar.gz # save image

docker load -i image_test.tar.gz # load image
```



### 4. 容器操作



##### 4.1 运行和进入容器

```bash
docker run -d -p 16097:16097 -p 15098:15098  image_name # 启动容器

docker extc -it 容器名字 bash # 进入容器内部
docker exec -it 容器ID  sh   # 进入容器内部
```



##### 4.2 看容器log

```bash
docker logs -f CONTAINER_ID

docker logs -f --tail=100 CONTAINER_ID
```



##### 4.3 容器和宿主拷贝内容

```bash
docker cp foo.txt mycontainer:/foo.txt 
docker cp mycontainer:/foo.txt foo.txt
```



##### 4.4 容器访问宿主主机端口

+ https://jingsam.github.io/2018/10/16/host-in-docker.html



### 5. 参考资料

+ https://blog.csdn.net/wdq347/article/details/78753322 docker之镜像制作
+ https://deepzz.com/post/dockerfile-best-practices.html  如何写好Dockerfile，Dockerfile最佳实践