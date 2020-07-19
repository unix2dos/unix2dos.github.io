---
title: io多路复用select_poll_epoll
tags: linux
abbrlink: 457c2d1f
categories:
  - 2-linux系统
  - 系统
date: 2017-05-01 17:54:46
---



# 1. IO分类

#### 1.1 同步IO

进程也可以换成线程

+ 阻塞IO (问一次 + 傻等)

  进程 在阻塞IO读 recvfrom 操作的两个阶段都是等待的。

  + 在数据没准备好的时候，process原地等待kernel准备数据。

  + kernel准备好数据后，process继续等待kernel将数据copy到自己的buffer。在kernel完成数据的copy后process才会从recvfrom系统调用中返回。

  <!-- more -->

+ 非阻塞IO (不停的催问 + 傻等)

  进程在非阻塞IO读 recvfrom 操作的第一个阶段是不会等待的。

  + 如果kernel数据还没准备好，那么recvfrom会立刻返回一个EWOULDBLOCK错误。

  + 当kernel准备好数据后，进入处理的第二阶段的时候，process会等待kernel将数据copy到自己的buffer，在kernel完成数据的copy后process才会从recvfrom系统调用中返回。

+ IO多路复用 (一个家长拦截 + 谁好了谁干活)

  在IO多路复用的时候，进程在两个处理阶段都是等待的。
  
  初看好像IO多路复用没什么用，其实select、poll、epoll的优势在于 **可以以较少的代价来同时监听处理多个IO。**



### 1.2 异步IO

异步IO要求进程 在recvfrom操作的两个处理阶段上都不能等待。

也就是process调用recvfrom后立刻返回，kernel自行去准备好数据并将数据从kernel的buffer中copy到process的buffer在通知process读操作完成了，然后process在去处理。

遗憾的是，linux的网络IO中是不存在异步IO的，linux的网络IO处理的第二阶段总是阻塞等待数据copy完成的。真正意义上的网络异步IO是Windows下的IOCP（IO完成端口）模型。

很多时候，我们比较容易混淆 非阻塞 IO和 异步 IO，其实它俩是有区别的。

+ 其实 非阻塞 IO仅仅要求处理的第一阶段不 block即可。
+ 异步 IO 要求两个阶段都不能block住。



### 1.3 总结

有人会说，非阻塞 IO并没有被block啊。这里有个非常“狡猾”的地方，定义中所指的”IO operation”是指真实的IO操作，就是例子中的recvfrom这个system call。

非阻塞 IO在执行recvfrom这个system call的时候，如果kernel的数据没有准备好，这时候不会block进程。

但是，当kernel中数据准备好的时候，recvfrom会将数据从kernel拷贝到用户内存中，这个时候进程是被block了，在这段时间内，进程是被block的。



# 2. IO多路复用

### 2.1 select 调用后阻塞, 等待文件描述符就绪返回, 然后遍历获取

`int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`

select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符。



当用户进程调用select的时候，select会将需要监控的 readfds 集合拷贝到内核空间（假设监控的仅仅是socket可读），然后遍历自己监控的socket sk，挨个调用sk的poll逻辑以便检查该sk是否有可读事件，遍历完所有的sk后，如果没有任何一个sk可读，那么select会调用schedule_timeout进入schedule循环，使得process进入睡眠。

如果在timeout时间内某个sk上有数据可读了，或者等待timeout了，则调用select的进程会被唤醒，接下来select就是遍历监控的sk集合，挨个收集可读事件并返回给用户了

1. 需要把文件描述符拷贝到内核
2. fds集合有限制 1024
3. 遍历效率低


### 2.2 poll 只解除了select1024的限制

`int poll(struct pollfd *fds, nfds_t nfds, int timeout);`

poll和select非常相似，poll并没着手解决性能问题，poll只是解决了select的问题 fds集合大小1024限制问题。所以是个鸡肋。

poll改变了fds集合的描述方式，使用了pollfd结构而不是select的fd_set结构，使得poll支持的fds集合限制远大于select的1024。

poll虽然解决了fds集合大小1024的限制问题，但是，它并没改变大量描述符数组被整体复制于用户态和内核态的地址空间之间，以及个别描述符就绪触发整体描述符集合的遍历的低效问题。

poll随着监控的socket集合的增加性能线性下降，poll不适合用于大并发场景。



### 2.3 epoll

+ `int epoll_create(int size)；`

  创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大  新版本用红黑树,这个参数意义不大了

+ `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；`

  + epfd：是epoll_create()的返回值。

  + op：表示op操作，分别添加、删除和修改对fd的监听事件。

    + 添加EPOLL_CTL_ADD，
    + 删除EPOLL_CTL_DEL，
    + 修改EPOLL_CTL_MOD。

  + fd：是需要监听的fd（文件描述符）

  + epoll_event：是告诉内核需要监听什么事，struct epoll_event结构如下：

    ```c
    struct epoll_event {
      __uint32_t events;  /* Epoll events */
      epoll_data_t data;  /* User data variable */
    };
    ```

    events可以是以下几个宏的集合：

    + EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
    + EPOLLOUT：表示对应的文件描述符可以写；
    + EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
    + EPOLLERR：表示对应的文件描述符发生错误；
    + EPOLLHUP：表示对应的文件描述符被挂断；
    + EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
    + EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里



+ `int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);`

  等待epfd上的io事件，最多返回maxevents个事件。

  参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size

  参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。

  该函数返回需要处理的事件数目，如返回0表示已超时。



##### 2.3.1 epoll拷贝问题的解决

epoll引入了epoll_ctl系统调用，将高频调用的epoll_wait和低频的epoll_ctl隔离开。同时，epoll_ctl通过(EPOLL_CTL_ADD、EPOLL_CTL_MOD、EPOLL_CTL_DEL)三个操作来分散对需要监控的fds集合的修改，做到了有变化才变更。

将select或poll高频、大块内存拷贝(集中处理)变成epoll_ctl的低频、小块内存的拷贝(分散处理)，避免了大量的内存拷贝。




同时，对于高频epoll_wait的可读就绪的fd集合返回的拷贝问题，epoll通过内核与用户空间mmap(内存映射)同一块内存来解决。mmap将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址（不管是用户空间还是内核空间都是虚拟地址，最终要通过地址映射映射到物理地址），使得这块物理内存对内核和对用户均可见，减少用户态和内核态之间的数据交换。



##### 2.3.2 epoll循环问题的解决

epoll引入了2个中间层，一个双向链表(ready_list)，一个单独的睡眠队列(single_epoll_wait_list)，



epoll巧妙的引入一个中间层解决了大量监控socket的无效遍历问题。epoll在中间层上为每个监控的socket准备了一个单独的回调函数epoll_callback_sk，而对于select/poll，所有的socket都公用一个相同的回调函数。正是这个单独的回调epoll_callback_sk使得每个socket都能单独处理自身，当自己就绪的时候将自身socket挂入epoll的ready_list。



同时，epoll引入了一个睡眠队列single_epoll_wait_list，分割了两类睡眠等待。进程不再睡眠在所有的socket的睡眠队列上，而是睡眠在epoll的睡眠队列上，在等待”任意一个socket可读就绪”事件。而中间wait_entry_sk则代替进程睡眠在具体的socket上，当socket就绪的时候，它就可以处理自身了。



##### 2.3.3 epoll LT ET

　epoll对文件描述符的操作有两种模式：LT（水平触发）和ET（边缘触发），LT模式是默认模式。

+ LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

  

  LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的。

  

+ ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

  

  ET(边缘触发)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个EWOULDBLOCK 错误）。

  

  但是请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once)
  
  
  
  ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。



当使用epoll的ET模型来工作时，当产生了一个EPOLLIN事件后，读数据的时候需要考虑的是当recv()返回的大小如果等于请求的大小，那么很有可能是缓冲区还有数据未读完，也意味着该次事件还没有处理完，所以还需要再次读取：(阻塞其他的)




### 2.4 总结

+ select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用`epoll_wait`不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在`epoll_wait`中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

+ select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次

  而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。

+ 如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用多线程 + 阻塞 IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。

+ 如果没有大量的idle -connection或者dead-connection，epoll的效率并不会比select/poll高很多，但是当遇到大量的idle- connection，就会发现epoll的效率大大高于select/poll。

+ 在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的进程其实是一直被block的。只不过进程是被select这个函数block，而不是被socket IO给block。



# 3. 问题总结

### 3.1  我在知乎的回答:

IO模式一般分为同步IO和异步IO.  同步IO会阻塞进程, 异步IO不会阻塞进程. 目前linux上大部分用的是同步IO, 异步IO在linux上目前还不成熟, 不过windows的iocp算是真正的异步IO。



同步IO又分为阻塞IO, 非阻塞IO, IO多路复用.  What? 同步IO明明会阻塞进程,为什么也包括非阻塞IO?  因为非阻塞IO虽然在请求数据时不阻塞, 但真正数据来临时,也就是内核数据拷贝到用户数据时, 此时进程是阻塞的.



那么这些IO模式的区别分别是什么? 接下来举个小例子来说明. 假设你现在去女生宿舍楼找自己的女神, 但是你只知道女神的手机号,并不知道女神的具体房间



先说同步IO的情况,

1. 阻塞IO,   给女神发一条短信, 说我来找你了, 然后就默默的一直等着女神下楼, 这个期间除了等待你不会做其他事情, 属于备胎做法.



2. 非阻塞IO, 给女神发短信, 如果不回, 接着再发, 一直发到女神下楼, 这个期间你除了发短信等待不会做其他事情, 属于专一做法.



3. IO多路复用,  是找一个宿管大妈来帮你监视下楼的女生, 这个期间你可以些其他的事情. 例如可以顺便看看其他妹子,玩玩王者荣耀, 上个厕所等等.  IO复用又包括 select, poll, epoll 模式. 那么它们的区别是什么?



3.1 select大妈    每一个女生下楼, select大妈都不知道这个是不是你的女神, 她需要一个一个询问, 并且select大妈能力还有限, 最多一次帮你监视1024个妹子



3.2 poll大妈不限制盯着女生的数量,  只要是经过宿舍楼门口的女生, 都会帮你去问是不是你女神



3.3 epoll大妈不限制盯着女生的数量, 并且也不需要一个一个去问.  那么如何做呢?  epoll大妈会为每个进宿舍楼的女生脸上贴上一个大字条,上面写上女生自己的名字,  只要女生下楼了, epoll大妈就知道这个是不是你女神了, 然后大妈再通知你.



上面这些同步IO有一个共同点就是, 当女神走出宿舍门口的时候, 你已经站在宿舍门口等着女神的, 此时你属于阻塞状态



接下来是异步IO的情况

你告诉女神我来了, 然后你就去王者荣耀了, 一直到女神下楼了, 发现找不见你了, 女神再给你打电话通知你, 说我下楼了, 你在哪呢?  这时候你才来到宿舍门口. 此时属于逆袭做法.



# 4. 代码

### 4.1 select

`server.c`

```c
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <sys/select.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <ctype.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "wrap.h"

#define PORT 8000
#define MAXLINE 1024
int main()
{
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int server_id = Socket(PF_INET, SOCK_STREAM, 0);

	struct sockaddr_in server, client;
	bzero(&server, sizeof(server));	
	server.sin_family = PF_INET;
	server.sin_port = htons(PORT);
	server.sin_addr.s_addr = htonl(INADDR_ANY);

  int opt = 1;
	setsockopt(server_id, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

	Bind(server_id, (struct sockaddr*)&server, sizeof(server));
	Listen(server_id, 20);
	printf("Accept connections...\n");

	int clients[FD_SETSIZE]; 
	for (int i = 0; i < FD_SETSIZE; ++i) {
		clients[i] = -1;
	}
	fd_set rset, allset;
	FD_ZERO(&allset);
	FD_SET(server_id, &allset);
	int maxfd = server_id;
	int maxi = -1;

	while (1) {
		rset = allset;
    // 只监听读描述符
		int iready = select(maxfd+1, &rset, NULL, NULL, NULL);
		if (iready < 0) {
			perr_exit("select error");
		}

		if (FD_ISSET(server_id, &rset)) {
			// 说明有新的 client 写
			socklen_t len = sizeof(client);
			int client_id = Accept(server_id, (struct sockaddr*)&client, &len);
			printf("received from %s at PORT %d\n",
					inet_ntop(PF_INET, &client.sin_addr, str, sizeof(str)),	
					ntohs(client.sin_port));

			int i = 0;
			for (; i < FD_SETSIZE; ++i) {
				if (clients[i] < 0) {
					clients[i] = client_id;
					break;
				}
			}
			if (i == FD_SETSIZE) {
				fputs("too many clients\n", stderr);
				exit(1);
			}
			FD_SET(client_id, &allset);
			if (client_id > maxfd) {
				maxfd = client_id;
			}
			if (i > maxi) {
				maxi = i;
			}
			if (--iready == 0) {
				continue;
			}
		}

		for (int i = 0; i <= maxi; ++i) {
			int fd = clients[i];
			if (fd < 0) {
				continue;	
			}
			if (FD_ISSET(fd, &rset)) {
				int n = Read(fd, buf, sizeof(buf));
				if (n == 0) {
					Close(fd);
					FD_CLR(fd, &allset);
					clients[i] = -1;
				} else {
					for (int i = 0; i < n; ++i) {
						buf[i] = toupper(buf[i]);
					}
					Write(fd, buf, n);
				}
				if (--iready == 0) {
					break;
				}
			}
		}
	}

	return 0;
}
```

`client.c`

```c
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <ctype.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "wrap.h"


#define PORT 8000
#define MAXLINE 1024
int main(int argc, char* agrv[])
{
	char buf[MAXLINE];
	memset(buf, 0, sizeof(buf));
	int server_id = Socket(PF_INET, SOCK_STREAM, 0);

	struct sockaddr_in server;
	bzero(&server, sizeof(server));	
	server.sin_family = PF_INET;
	server.sin_port = htons(PORT);
	inet_pton(PF_INET, "127.0.0.1", &server.sin_addr);

	Connect(server_id, (struct sockaddr*)&server, sizeof(server));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(server_id, buf, strlen(buf));
		int n = Read(server_id, buf, MAXLINE);
		if (n == 0) {
			printf("the other side has been closed.\n");
		} else {
			Write(STDOUT_FILENO, buf, n);
		}
	}
	Close(server_id);
	return 0;
}
```



### 4.2 poll

`server.c`

```c
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <sys/select.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <ctype.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <poll.h>
#include "wrap.h"

#define PORT 8000
#define MAXLINE 1024
#define OPEN_MAX 1000
int main()
{
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int server_id = Socket(PF_INET, SOCK_STREAM, 0);

	struct sockaddr_in server, client;
	bzero(&server, sizeof(server));	
	server.sin_family = PF_INET;
	server.sin_port = htons(PORT);
	server.sin_addr.s_addr = htonl(INADDR_ANY);
	
	int opt = 1;
	setsockopt(server_id, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

	Bind(server_id, (struct sockaddr*)&server, sizeof(server));
	Listen(server_id, 20);
	printf("Accept connections...\n");


	struct pollfd clients[OPEN_MAX];
	clients[0].fd = server_id;
	clients[0].events = POLLIN;
	for (int i = 1; i < OPEN_MAX; i++) {
		clients[i].fd = -1;
	}

	int maxi = 0;
	while (1) {
    // 监听 POLLIN 事件
		int iready = poll(clients, maxi+1, -1);	
		if (iready < 0) {
			perr_exit("poll error");
		}
		
    // 说明 client 来了写
		if (clients[0].revents & POLLIN) {
			socklen_t len = sizeof(client);
			int client_id = Accept(server_id, (struct sockaddr*)&client, &len);	
			printf("received from %s at PORT %d\n",
					inet_ntop(PF_INET, &client.sin_addr, str, sizeof(str)),
					ntohs(client.sin_port));

			int i = 1;
			for (; i < OPEN_MAX; ++i) {
				if (clients[i].fd < 0) {
					clients[i].fd = client_id;	
					break;
				}
			}

			if (i == OPEN_MAX) {
				fputs("too many clients\n", stderr);
				exit(1);
			}

			clients[i].events = POLLIN;
			if (i > maxi) {
				maxi = i;
			}
			if (--iready == 0) {
				continue;
			}
		}

		for (int i = 1; i <= maxi; ++i) {
			if (clients[i].fd < 0) {
				continue;
			}	

			if (clients[i].revents & POLLIN) {
				int n = Read(clients[i].fd, buf, sizeof(buf));
				if (n == 0) {
					Close(clients[i].fd);
					clients[i].fd = -1;
				} else {
					for (int i = 0; i < n; ++i) {
						buf[i] = toupper(buf[i]);
					}
					Write(clients[i].fd, buf, n);
				}
				if (--iready == 0) {
					break;
				}
			}
		}
	}
	return 0;
}
```



`client.c`

```c
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <ctype.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "wrap.h"


#define PORT 8000
#define MAXLINE 1024
int main(int argc, char* agrv[])
{
	char buf[MAXLINE];
	memset(buf, 0, sizeof(buf));
	int server_id = Socket(PF_INET, SOCK_STREAM, 0);

	struct sockaddr_in server;
	bzero(&server, sizeof(server));	
	server.sin_family = PF_INET;
	server.sin_port = htons(PORT);
	inet_pton(PF_INET, "127.0.0.1", &server.sin_addr);

	Connect(server_id, (struct sockaddr*)&server, sizeof(server));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(server_id, buf, strlen(buf));
		int n = Read(server_id, buf, MAXLINE);
		if (n == 0) {
			printf("the other side has been closed.\n");
		} else {
			Write(STDOUT_FILENO, buf, n);
		}
	}
	Close(server_id);
	return 0;
}
```



### 4.3 epoll

`server.c`

```c
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/uio.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <ctype.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include "wrap.h"

#define PORT 8000
#define MAXLINE 1024
#define OPEN_MAX 1000

void add_event(int epollid, int fd, int state)
{
	struct epoll_event ev;
	ev.data.fd = fd;
	ev.events = state;
	epoll_ctl(epollid, EPOLL_CTL_ADD, fd, &ev);
}
void modify_event(int epollid, int fd, int state)
{
	struct epoll_event ev;
	ev.data.fd = fd;
	ev.events = state;
	epoll_ctl(epollid, EPOLL_CTL_MOD, fd, &ev);
}
void delete_event(int epollid, int fd, int state)
{
	struct epoll_event ev;
	ev.data.fd = fd;
	ev.events = state;
	epoll_ctl(epollid, EPOLL_CTL_DEL, fd, &ev);
}


int main()
{
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int server_id = Socket(PF_INET, SOCK_STREAM, 0);

	struct sockaddr_in server, client;
	bzero(&server, sizeof(server));	
	server.sin_family = PF_INET;
	server.sin_port = htons(PORT);
	server.sin_addr.s_addr = htonl(INADDR_ANY);
	
	int opt = 1;
	setsockopt(server_id, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

	Bind(server_id, (struct sockaddr*)&server, sizeof(server));
	Listen(server_id, 20);
	printf("Accept connections...\n");


	struct epoll_event events[EPOLLEVENTS];
	int epollfd = epoll_create(FDSIZE);

	struct epoll_event ev;
	ev.events = EPOLLIN;
	ev.data.fd = STDIN_FILENO;
	epoll_ctl(epollfd, EPOLL_CTL_ADD, STDIN_FILENO, &ev);

	while (1) {
		int ret = epoll_wait(epollfd, events, EPOLLEVENTS, -1);
		for (int i = 0; i < ret; ++i) {
			int fd = events[i].data.fd;
			if (fd == server_id && (events[i].events & EPOLLIN)) {
				socklen_t len = sizeof(client);
				int client_id = Accept(server_id, (struct sockaddr*)&client, &len);	
				printf("received from %s at PORT %d\n",
						inet_ntop(PF_INET, &client.sin_addr, str, sizeof(str)),
						ntohs(client.sin_port));

				struct epoll_event ev;
				ev.events = state;
				ev.data.fd = fd;
				epoll_ctl(epollfd,EPOLL_CTL_ADD,fd,&ev);

			} else if (events[i].events & EPOLLIN) {
				int n = Read(clients[i].fd, buf, sizeof(buf));
				if (n == 0) {
					Close(clients[i].fd);

					struct epoll_event ev;
					ev.events = EPOLLIN;
					ev.data.fd = fd;
					epoll_ctl(epollfd,EPOLL_CTL_DEL,fd,&ev);

				} else {
					struct epoll_event ev;
					ev.events = EPOLLOUT;//由读改为写
					ev.data.fd = fd;
					epoll_ctl(epollfd,EPOLL_CTL_MOD,fd,&ev);
				}

			} else if (events[i].events & EPOLLOUT) {
				for (int i = 0; i < n; ++i) {
					buf[i] = toupper(buf[i]);
				}
				int n = Write(fd, buf, n);
				if (n < 0) {
					struct epoll_event ev;
					ev.events = EPOLLIN;
					ev.data.fd = fd;
					epoll_ctl(epollfd,EPOLL_CTL_DEL,fd,&ev);

				} else {
					struct epoll_event ev;
					ev.events = EPOLLIN;//由写改为读
					ev.data.fd = fd;
					epoll_ctl(epollfd,EPOLL_CTL_MOD,fd,&ev);
				}
			}
		}
	}

	Close(epollfd);


	return 0;
}
```



`client.c`

```c
#include <string.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include "wrap.h"
#include "epollUtil.h"

#define IP "127.0.0.1"
#define PORT 8000
#define FD_SIZE 1024
#define EPOLLEVENTS 20
int main(int agrc, char* argv[]) {
	char buf[1024];
	memset(buf, 0, sizeof(buf));

	struct sockaddr_in server;
	bzero(&server, sizeof(server));
	server.sin_family = AF_INET;
	server.sin_port = htons(PORT);
	inet_pton(AF_INET, IP, &server.sin_addr);
	

	int server_id = Socket(AF_INET, SOCK_STREAM, 0);
	Connect(server_id, (struct sockaddr*)&server, sizeof(server));

	struct epoll_event events[EPOLLEVENTS];
	int epollfd = epoll_create(FD_SIZE);
	add_event(epollfd, STDIN_FILENO, EPOLLIN);
	while (1) {
	
		int ret = epoll_wait(epollfd, events, EPOLLEVENTS, -1);
		for (int i = 0; i < ret; ++i) {
			int fd = events[i].data.fd;

			if (events[i].events & EPOLLIN) {
				int n = Read(fd, buf, sizeof(buf));	
				if (n == 0) {
					Close(fd);	
				} else {
					if (fd == STDIN_FILENO) {
						add_event(epollfd, server_id, EPOLLOUT);
					} else {
						delete_event(epollfd, server_id, EPOLLIN);	
						add_event(epollfd, STDOUT_FILENO, EPOLLOUT);
					}	
				}
			} else if (events[i].events & EPOLLOUT) {
				Write(fd, buf, strlen(buf));	
				if (fd == STDOUT_FILENO) {
					delete_event(epollfd, fd, EPOLLOUT);
				} else {
					modify_event(epollfd, fd, EPOLLIN);
				}
			}
		}
	}

	Close(server_id);
	return 0;
}
```



### 4.4 总结

+ select 

  死循环里用 select 阻塞, 返回后开始遍历

+ poll

  死循环里用 poll 阻塞, 返回后开始遍历

+ epoll

  死循环里用 epoll_wait 阻塞

  