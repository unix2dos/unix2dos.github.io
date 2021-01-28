---
title: "systemctl详解"
date: 2021-01-28 00:00:00
tags:
- systemctl
- linux
---

# 1. 历史

历史上，Linux 的启动一直采用init进程。 Systemd 设计目标是，为系统的启动和管理提供一套完整的解决方案。
根据 Linux 惯例，字母d是守护进程（daemon）的缩写。 Systemd 这个名字的含义，就是它要守护整个系统。
使用了 Systemd，就不需要再用init了。Systemd 取代了initd，成为系统的第一个进程（PID 等于 1），其他进程都是它的子进程。

<!-- more -->


CentOS 7 使用 Systemd 替换了SysV,  Ubuntu 从 15.04 开始使用 Systemd.

在 SysV-init 时代，将每个服务项目编号，依次执行启动脚本。Ubuntu 的 Upstart 解决了没有直接依赖的启动之间的并行启动。而 Systemd 通过 Socket 缓存、DBus 缓存和建立临时挂载点等方法进一步解决了启动进程之间的依赖，做到了所有系统服务并发启动。对于用户自定义的服务，Systemd 允许配置其启动依赖项目，从而确保服务按必要的顺序运行。



CGroup 提供了类似文件系统的接口，当进程创建子进程时，子进程会继承父进程的 CGroup。因此无论服务如何启动新的子进程，所有的这些相关进程都会属于同一个 CGroup
systemd 通过 CGroup 不仅能够实现服务之间访问隔离，限制特定应用程序对系统资源的访问配额，还能更精确地管理服务的生命周期



# 2. Unit

Systemd 可以管理所有系统资源：将系统资源划分为12类。将每个系统资源称为一个 Unit。

Unit 是 Systemd 管理系统资源的基本单位。使用一个 Unit File 作为 Unit 的单元文件，Systemd 通过单元文件控制 Unit 的启动。

例如，MySQL服务被 Systemd 视为一个 Unit，使用一个 mysql.service 作为启动配置文件

### 2.1 Unit File（单元文件|配置文件）

Systemd 将系统资源划分为12类，对应12种类型的单元文件

| 系统资源类型 | 单元文件扩展名 | 单元文件描述                                                 |
| ------------ | -------------- | ------------------------------------------------------------ |
| Service      | .service       | 封装守护进程的启动、停止、重启和重载操作，是最常见的一种 Unit 文件 |
| Target       | .target        | 定义 target 信息及依赖关系，一般仅包含 Unit 段               |
| Device       | .device        | 对于 `/dev` 目录下的硬件设备，主要用于定义设备之间的依赖关系 |
| Mount        | .mount         | 定义文件系统的挂载点，可以替代过去的 `/etc/fstab` 配置文件   |
| Automount    | .automount     | 用于控制自动挂载文件系统，相当于 SysV-init 的 autofs 服务    |
| Path         | .path          | 用于监控指定目录或文件的变化，并触发其它 Unit 运行           |
| Scope        | .scope         | 这种 Unit 文件不是用户创建的，而是 Systemd 运行时产生的，描述一些系统服务的分组信息 |
| Slice        | .slice         | 用于表示一个 CGroup 的树                                     |
| Snapshot     | .snapshot      | 用于表示一个由 systemctl snapshot 命令创建的 Systemd Units 运行状态快照，可以切回某个快照 |
| Socket       | .socket        | 监控来自于系统或网络的数据消息                               |
| Swap         | .swap          | 定义一个用户做虚拟内存的交换分区                             |
| Timer        | .timer         | 用于配置在特定时间触发的任务，替代了 Crontab 的功能          |

对于操作单元文件的命令，如果缺省扩展名，则默认`.service`扩展名

### 2.2 语法

先看一个示例

```
[Unit]
Description=lw-music
After=network.target
[Service]
ExecStart=/usr/bin/node /opt/liuwei/UnblockNeteaseMusic/app.js -e https://music.liuvv.com -s -p 8080:8081
Restart=always
RestartSec=5
[Install]
WantedBy=default.target
```

Unit 文件可以分为三个配置区段：

+ Unit 段：所有 Unit 文件通用，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系
+ Service 段：服务（Service）类型的 Unit 文件（后缀为 .service）特有的，用于定义服务的具体管理和执行动作
+ Install 段：所有 Unit 文件通用，用来定义如何启动，以及是否开机启动


单元文件中的区段名和字段名大小写敏感

每个区段内都是一些等号连接的键值对（键值对的等号两侧不能有空格）



### 2.3 Unit 段

- `Description`：当前服务的简单描述
- `Documentation`：文档地址，可以是一个或多个文档的 URL 路径
- `Requires`：与其它 Unit 的强依赖关系，如果其中任意一个 Unit 启动失败或异常退出，当前 Unit 也会被退出
- `Wants`：与其它 Unit 的弱依赖关系，如果其中任意一个 Unit 启动失败或异常退出，不影响当前 Unit 继续执行
- `After`：该字段指定的 Unit 全部启动完成以后，才会启动当前 Unit
- `Before`：该字段指定的 Unit 必须在当前 Unit 启动完成之后再启动
- `Binds To`：与 Requires 相似，该字段指定的 Unit 如果退出，会导致当前 Unit 停止运行
- `Part Of`：一个 Bind To 作用的子集，仅在列出的 Unit 失败或重启时，终止或重启当前 Unit，而不会随列出Unit 的启动而启动



### 2.4 Service段

所有的启动设置之前，都可以加上一个连词号（`-`），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。比如，`EnvironmentFile=-/etc/sysconfig/sshd`（注意等号后面的那个连词号），就表示即使`/etc/sysconfig/sshd`文件不存在，也不会抛出错误。

##### 2.4.1 启动类型
+ Type：定义启动时的进程行为。它有以下几种值。

  **Type=simple**：默认值，ExecStart字段启动的进程为主进程
  服务进程不会 fork，如果该服务要启动其他服务，不要使用此类型启动，除非该服务是 socket 激活型

  **Type=forking**：ExecStart字段将以fork()方式从父进程创建子进程启动，创建后父进程会立即退出，子进程成为主进程。
  通常需要指定PIDFile字段，以便 Systemd 能够跟踪服务的主进程

  对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求，使用此类型启动即可

  **Type=oneshot**：只执行一次，Systemd 会等当前服务退出，再继续往下执行, 适用于只执行一项任务、随后立即退出的服务
  通常需要指定RemainAfterExit=yes字段，使得 Systemd 在服务进程退出之后仍然认为服务处于激活状态

  **Type=dbus**：当前服务通过 D-Bus 信号启动。当指定的 BusName 出现在 DBus 系统总线上时，Systemd认为服务就绪

  **Type=notify**：当前服务启动完毕会发出通知信号，通知 Systemd，然后 Systemd 再启动其他服务

  **Type=idle**：Systemd 会等到其他任务都执行完，才会启动该服务。一种使用场合是：让该服务的输出，不与其他服务的输出相混合
##### 2.4.2 启动行为

+ `ExecStart`：启动当前服务的命令

  ```bash
  ExecStart=/bin/echo execstart1
  ExecStart=
  ExecStart=/bin/echo execstart2
  ```

  顺序执行设定的命令，把字段置空，表示清除之前的值

- `ExecStartPre`：启动当前服务之前执行的命令
- `ExecStartPost`：启动当前服务之后执行的命令
- `ExecReload`：重启当前服务时执行的命令
- `ExecStop`：停止当前服务时执行的命令
- `ExecStopPost`：停止当前服务之后执行的命令
- `RemainAfterExit`：当前服务的所有进程都退出的时候，Systemd 仍认为该服务是激活状态, 这个配置主要是提供给一些并非常驻内存，而是启动注册后立即退出，然后等待消息按需启动的特殊类型服务使用的

+ `TimeoutSec`：定义 Systemd 停止当前服务之前等待的秒数

##### 2.4.3 重启行为

+ `RestartSec`：Systemd 重启当前服务间隔的秒数
+ `KillMode`：定义 Systemd 如何停止服务，可能的值包括：
  control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉
  process：只杀主进程（sshd 服务，推荐值）
  mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
  none：没有进程会被杀掉，只是执行服务的 stop 命令。
+ `Restart`：定义何种情况 Systemd 会自动重启当前服务，可能的值包括：
  no（默认值）：退出后不会重启
  on-success：只有正常退出时（退出状态码为0），才会重启
  on-failure：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启（守护进程，推荐值）
  on-abnormal：只有被信号终止和超时，才会重启（对于允许发生错误退出的服务，推荐值）
  on-abort：只有在收到没有捕捉到的信号终止时，才会重启
  on-watchdog：超时退出，才会重启
  always：不管是什么退出原因，总是重启

##### 2.4.4 上下文

- `PIDFile`：指向当前服务 PID file 的绝对路径。

- `User`：指定运行服务的用户

- `Group`：指定运行服务的用户组

- `EnvironmentFile`：指定当前服务的环境参数文件。该文件内部的`key=value`键值对，可以用`$key`的形式，在当前配置文件中获取

  启动`sshd`，执行的命令是`/usr/sbin/sshd -D $OPTIONS`，其中的变量`$OPTIONS`就来自`EnvironmentFile`字段指定的环境参数文件。

### 2.5 Install段

- `WantedBy`：它的值是一个或多个 target，执行enable命令时，符号链接会放入`/etc/systemd/system`目录下以 target 名 + `.wants`后缀构成的子目录中

- `RequiredBy`：它的值是一个或多个 target，执行enable命令时，符号链接会放入`/etc/systemd/system`目录下以 target 名 + `.required`后缀构成的子目录中

- `Alias`：当前 Unit 可用于启动的别名

- `Also`：当前 Unit 被 enable/disable 时，会被同时操作的其他 Unit

  

### 2.6 占位符

在 Unit 文件中，有时会需要使用到一些与运行环境有关的信息，例如节点 ID、运行服务的用户等。这些信息可以使用占位符来表示，然后在实际运行中动态地替换为实际的值。



### 2.7 模板

在现实中，往往有一些应用需要被复制多份运行，就会用到模板文件

模板文件的写法与普通单元文件基本相同，只是模板文件名是以 @ 符号结尾。例如：apache@.service

通过模板文件启动服务实例时，需要在其文件名的 @ 字符后面附加一个用于区分服务实例的参数字符串，通常这个参数是用于监控的端口号或控制台 TTY 编译号

```
systemctl start apache@8080.service
```

Systemd 在运行服务时，首先寻找跟单元名完全匹配的单元文件，如果没有找到，才会尝试选择匹配模板

例如上面的命令，System 首先会在约定的目录下寻找名为 apache@8080.service 的单元文件，如果没有找到，而文件名中包含 @ 字符，它就会尝试去掉后缀参数匹配模板文件。对于 apache@8080.service，Systemd 会找到 apache@.service 模板文件，并通过这个模板文件将服务实例化。



# 3. Target

如果一个target只包含一个Unit，那么该 target，没有对应的目录，指的就是这个 Unit, 例如 `hibernate.target`只包含 `systemd-hibernate.service`一个Unit.

如果一个target包含多个Unit，那么该target，有对应的 xxx.target.wants 目录，指的是目录里面所有的Unit, 例如`multi-user.target` 包含位于`/etc/systemd/system/multi-user.target.wants`目录下的多个 Unit.

### 3.1 含义

+ 系统的某个状态称为一个 target（类似于"状态点"）

  例如执行`systemd suspend`命令让系统暂停，会触发启动`suspend.target`，然后执行里面的`systemd-suspend.service` Unit，使系统达到一个暂停的状态

+ 达到某个系统状态，所需的一个或多个资源（Unit）称为一个 target（一个 Unit 组）

Systemd 使用 target 来划分和管理资源（Unit），启动（激活）某个 xxx.target 单元文件，通过执行该 target 包含的 Unit，使系统达到某种状态

### 3.2 启动 target

runlevel是 SysV init 初始化系统中的概念，在Systemd初始化系统中使用的是 Target，他们之间的映射关系是

| Runlevel | Target            | 说明                               |
| -------- | ----------------- | ---------------------------------- |
| 0        | poweroff.target   | 关闭系统                           |
| 1        | rescue.target     | 维护模式                           |
| 2,3,4    | multi-user.target | 多用户，无图形系统（命令行界面）   |
| 5        | graphical.target  | 多用户，图形化系统（图形用户界面） |
| 6        | reboot.target     | 重启系统                           |

### 3.3 启动过程

1. 读入 `/boot` 目录下的内核文件
2. 内核文件加载完之后，开始执行第一个程序`/sbin/init` 初始化进程，由 Systemd 初始化系统引导，完成相关的初始化工作
3. Systemd 执行`default.target` ，获知设定的启动 target (查看默认 target: `systemctl get-default)`
4. Systemd 执行启动 target 对应的单元文件。根据单元文件中定义的[依赖关系](https://www.freedesktop.org/software/systemd/man/bootup.html#System Manager Bootup)，传递控制权，依次执行其他 target 单元文件，同时启动每个 target 包含的单元



# 4. systemd

### 4.1 目录和文件

+ `/run/systemd/system/` 单元（服务）运行时生成的配置文件所在目录
+ `/etc/systemd/system/` 系统或用户自定义的配置文件，初始化过程中`Systemd`只执行`/etc/systemd/system`目录里面的配置文件
+ `/lib/systemd/system/`软件安装时添加的配置文件，类似于`/etc/init.d/`

  对于支持 Systemd 的程序，安装的时候，会自动的在 `/lib/systemd/system` 目录添加一个配置文件

+ `/etc/systemd/system/default.target ` Systemd 执行的第一个单元文件，符号链接到默认启动 target 对应的 `.target` 单元文件

### 4.2 优先级

SysV 的启动脚本放在`/etc/init.d`目录下

Systemd 的单元文件放在`/etc/systemd/system` 和 `/lib/systemd/system`目录下

当一个程序在3个目录下都存在启动方式时，优先级是`/etc/systemd/system --> /lib/systemd/system --> /etc/init.d`



### 4.3 系统管理命令

+ systemctl reboot 重启系统（异步操作）
+ systemctl poweroff 关闭系统，切断电源（异步操作）
+ systemctl halt 仅CPU停止工作，其他硬件仍处于开机状态（异步操作）
+ systemctl suspend 暂停系统（异步操作）将触发执行`suspend.target`
+ systemctl hibernate 让系统进入冬眠状态（异步操作）将触发执行`hibernate.target`



### 4.4 单元命令

+ systemctl list-units  列出当前已加载的单元（内存）,默认情况下仅显示处于激活状态（正在运行）的单元

+ systemctl start 启动单元

+ systemctl stop**停止单元

+ systemctl kill 杀掉单元进程

+ systemctl reload 不终止单元，重新加载 针对该单元的 运行配置文件，而不是 针对 systemd的 该单元的启动配置文件

+ systemctl restart 重启单元, 该单元在重启之前拥有的资源不会被完全清空，比如文件描述符存储设施

+ systemctl status 显示单元或进程所属单元的运行信息

+ systemctl is-active判断指定的单元是否处于激活状态

+ systemctl is-failed判断指定的单元是否处于启动失败状态

+ systemctl list-dependencies查看单元之间的依赖关系

+ systemctl show显示单元所有底层参数

+ systemctl isolate 切换到某个 target（系统状态），立即停止该 target 未包含的单元进程。也可以理解为切换 runlevel

+ systemctl cat 显示单元配置文件的备份文件



### 4.5 单元文件命令

+ systemctl list-unit-files, 列出所有已安装的单元文件和它们的启用状态

  将会列出文件的 state，包括 static, enabled, disabled, masked, indirect

  masked, service软链接到`/dev/null`, 该单元文件被禁止建立启动链接

  static, 该单元文件没有`[Install]`部分（无法执行），只能作为其他配置文件的依赖

  enabled, 已建立启动链接

  disabled, 没建立启动链接

+ systemctl enable, 使某个单元开机自启动, 这会根据单元文件内容中的`[Install]`指定的 target 组，创建一个软链接

+ systemctl disable, 取消某个单元开机自启动设置，删除软链接, 这会删除所有指向该单元文件的软链接，不仅仅是 enable 创建的

+ systemctl reenable, disable 和 enable 的结合，根据单元文件内容中的 [Install] 段，重置软链接

+ systemctl is-enabled, 检查某个单元是否是开机自启动的（建立的启动链接）

+ systemctl get-default, 获取默认启动 target，default-target 是指向该 target 的软链接

+ systemctl set-default, 设置默认启动 target，同时修改 default-target 指向设定的 target

+ systemctl daemon-reload, 重新加载所有的单元文件和依赖关系,对单元文件有修改的时候，需要执行该命令重新加载文件内容



### 4.6 systemctl 与 service 命令的区别

1. systemctl 融合了 service 和 chkconfig 的功能
2. 在 Ubuntu18.04 中没有自带 chkconfig 命令；service 命令实际上重定向到 systemctl 命令

| 动作                 | SysV Init 指令                 | Systemd 指令                              |
| -------------------- | ------------------------------ | ----------------------------------------- |
| 启动某服务           | service httpd start            | systemctl start httpd                     |
| 停止某服务           | service httpd stop             | systemctl stop httpd                      |
| 重启某服务           | service httpd restart          | systemctl restart httpd                   |
| 检查服务状态         | service httpd status           | systemctl status httpd                    |
| 删除某服务           | chkconfig --del httpd          | 停掉应用，删除其配置文件                  |
| 使服务开机自启动     | chkconfig --level 5 httpd on   | systemctl enable httpd                    |
| 使服务开机不自启动   | chkconfig --level 5 httpd off  | systemctl disable httpd                   |
| 查询服务是否开机自启 | chkconfig --list \| grep httpd | systemctl is-enabled httpd                |
| 加入自定义服务       | chkconfig --add test           | systemctl load test                       |
| 显示所有已启动的服务 | chkconfig --list               | systemctl list-unit-files \| grep enabled |



# 5. system 工具集

### 5.1 hostnamectl 

主机名管理命令

`hostnamectl status`

```
   Static hostname: tencent2
   Pretty hostname: VM-0-8-ubuntu
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4211ff3f594041f3966d836585a11a05
           Boot ID: 7a5ad9b3ae6640708a272e9a9d5f354d
    Virtualization: kvm
  Operating System: Ubuntu 18.04.4 LTS
            Kernel: Linux 4.15.0-88-generic
      Architecture: x86-64
```



# 6. 参考资料

+ https://www.cnblogs.com/usmile/p/13065594.html
+ https://cloud.tencent.com/developer/article/1516125
+ http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html
+ http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html
+ http://www.jinbuguo.com/systemd/systemd.html