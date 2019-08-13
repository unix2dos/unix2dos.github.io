---
title: ansible实践
tags:
  - ansible
  - linux
abbrlink: 18605da6
categories:
  - linux系统运维
date: 2019-08-01 19:54:46
---



### 1. ansible 安装

在 ansible 的世界里，我们会通过 inventory 档案来定义有哪些 managed node (被控端)，并借由 ssh 和 python 进行沟通。换句话说，当 control machine (主控端) 可以用 ssh 连上 managed node，且被连上的机器里有预载 python 时，ansile 就可以运作了.

+ 控制端

```bash
sudo apt install ansible #linux
brew install ansible # mac
```

+ 被控端

  要安装 python, 并且能被控制端 ssh 

<!-- more -->

### 2. ansible 配置

ansible的默认配置文件路径为 /etc/ansible，然而，一个常见的用途是将其安装在一个virtualenv中，在这种情况下，我们一般不会使用这些默认文件。我们可以根据需要在本地目录中创建配置文件。

##### 2.1 inventory文件

您可以创建一个inventory文件，用于定义将要管理的服务器。这个文件可以命名为任何名字，但我们通常会命名为hosts或者项目的名称。 

在hosts文件中，我们可以定义一些要管理的服务器。这里我们将定义我们可能要在“web”标签下管理的两个服务器。标签是任意的。

```bash
[web]
192.168.22.10
192.168.22.11
```

现在，让我们将hosts文件设置为指向本地主机local和remote虚拟远程主机。 

```bash
[local]
127.0.0.1

[remote]
192.168.1.2
```

### 3. ansible 使用

我们开始对服务器运行任务。ansible会假定你的服务器具有ssh访问权限，通常基于ssh-key。因为ansible使用ssh，所以它需要能够ssh连接到服务器。但是，ansible将尝试以正在运行的当前用户身份进行连接。如果我正在运行ansible的用户是ubuntu，它将尝试以ubuntu连接其他服务器。

Note: 控制端和被控端的用户很显然会不一样。

```bash
whoami # levonfly

ansible -i ./hosts --connection=local local -m ping
127.0.0.1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}


ansible -i ./hosts remote -m ping
192.168.1.2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 192.168.1.2 port 22: Operation timed out\r\n",
    "unreachable": true
}
```



+ 使用–connection=local告诉ansible不尝试通过ssh运行命令，因为我们只是影响本地主机。但是，我们仍然需要一个hosts文件，告诉我们连接到哪里。 

+ 在任何情况下，我们可以看到从ansible得到的输出是一些json，它告诉我们task（我们对ping模块的调用）是否进行了任何更改和结果。



命令说明：

```bash
-i ./hosts 				# 设置库存文件，命名为 hosts
remote，local，all # 使用这个标签的下定义的服务器hosts清单文件。“all”是针对文件中定义的每个服务器运行的特殊关键字, 注意all比较特殊
-m ping # 使用“ping”模块，它只是运行ping命令并返回结果
-c local| --connection=local # 在本地服务器上运行命令，而不是SSH

一些常用命令：
-i PATH --inventory=PATH # 指定host文件的路径，默认是在/etc/ansible/hosts
--private-key=PRIVATE_KEY_FILE_PATH # 使用指定路径的秘钥建立认证连接
-m DIRECTORY --module-path=DIRECTORY #指定module的目录来加载module，默认是/usr/share/ansible
-c CONNECTION --connection=CONNECTION #指定建立连接的类型，一般有ssh ，local
```



##### 3.1 模块（modules）

ansible使用“模块”来完成大部分的任务。模块可以做安装软件，复制文件，使用模板等等。

如果我们没有模块，我们将运行任意的shell命令，我们也可以使用bash脚本。这是一个任意shell命令看起来像在ansible（它使用的shell模块！）：

```bash
# 在本地执行ls命令
ansible -i ./hosts local --connection=local -m shell -a 'ls'

127.0.0.1 | SUCCESS | rc=0 >>
README.md
hosts
nginx.yml
roles

# 在远程安装nginx, 注意--become-user=root是改变控制端的用户, 不是被控端
ansible -i ./hosts remote -b --become-user=root -m shell -a 'yum install nginx'
172.24.120.46 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: liuwei@172.24.120.46: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).\r\n",
    "unreachable": true
}
```

命令说明:

```bash
-b # “成为”，在运行命令时告诉可以成为另一个用户。
--become-user=root # 以用户“root”运行以下命令, 是改变控制端的用户, 不是被控端!!!

-a #用于将任何参数传递给-m定义的模块
```



要在centos服务器上安装软件，“yum”模块将运行相同的命令，但确保幂等。

```bash
# 指定root用户yum安装nginx
ansible -i ./hosts remote -v -m yum -a 'name=nginx state=installed update_cache=true' -u root -k

No config file found; using defaults
SSH password: xxx
172.24.120.46 | SUCCESS => {
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
        "1:nginx-1.12.2-3.el7.x86_64 providing nginx is already installed"
    ]
}


# 指定普通用户yum安装nginx, 并且输入sudo密码
ansible -i ./hosts remote -v -m yum -a 'name=nginx state=installed update_cache=true' -u liuwei -k -s -K 
SSH password: xxx
SUDO password[defaults to SSH password]: xxx
172.24.120.46 | SUCCESS => {
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
        "1:nginx-1.12.2-3.el7.x86_64 providing nginx is already installed"
    ]
}
```

命令说明:

```bash
-v   	 # verbose mode, (-vvv for more, -vvvv to enable connection debugging)
-m apt # 使用apt模块
-a 'name=nginx state=installed update_cache=true' # 提供apt模块的参数，包括软件包名称，所需的结束状态以及是否更新软件包存储库缓存

常用命令：
-u USERNAME | --user=USERNAME #指定被控端的执行用户
-k | --ask-pass  #提示输入ssh的密码，而不是使用基于ssh的密钥认证

-s | --sudo  #指定用户的时候，使用sudo获得root权限
-K | --ask-sudo-pass #提示输入sudo密码，与--sudo一起使用
```

这将使用yum模块来更新存储库缓存并安装nginx（如果没有安装）。 运行任务的结果是”changed”: false。这表明没有变化; 我已经使用该shell模块安装了nginx 。好的是，我可以一遍又一遍地运行这个命令，而不用担心它会改变预期的结果 - nginx已经安装，ansible知道，并且不尝试重新安装它。 



##### 3.2 剧本（playbooks）

playbook可以运行多个任务，并提供一些更高级的功能。让我们将上述任务移到一本剧本中。在ansible中剧本（playbooks）和角色（roles）都使用yaml文件定义。 

nginx.yml：(通过yaml写所需参数)

```yaml
- hosts: remote
  become: yes
  become_user: root
  tasks:
   - name: Install Nginx
     yum:
       name: nginx
       state: installed
       update_cache: true
```



这将使用inventory文件中[remote]标签下的服务器hosts。在我们的tasks文件中使用become并become_user再次使用ansible来sudo以root用户身份运行命令，然后传递playbook文件。使用一个yaml playbook文件，我们需要使用这个ansible-playbook命令：



```bash
ansible-playbook -i ./hosts nginx.yml -k -K
SSH password:
SUDO password[defaults to SSH password]:

PLAY [remote] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [172.24.120.46]

TASK [Install Nginx] ********************************************************************************************************************************************************************
ok: [172.24.120.46]

PLAY RECAP ******************************************************************************************************************************************************************************
172.24.120.46              : ok=2    changed=0    unreachable=0    failed=0
```

我们在运行过程中获得了一些有用的反馈，包括“可执行任务”运行及其结果。在这里我们看到所有运行都ok，但没有改变。



##### 3.3 处理程序（handlers）

处理程序与任务完全相同（它可以做task可以做的任何事），但只有当另一个任务调用它时才会运行。您可以将其视为事件系统的一部分; 处理程序将通过其侦听的事件调用进行操作。 这对于运行任务后可能需要的“辅助”操作非常有用，例如在配置更改后安装或重新加载服务后启动新服务。

```yaml
- hosts: remote
  become: yes
  become_user: root
  tasks:
   - name: Install Nginx
     yum:
       name: nginx
       state: installed
       update_cache: true
     notify: #复制的时候, 要注意空格, 对齐
      - Start Nginx

  handlers:
   - name: Start Nginx
     service:
       name: nginx
       state: started
```

这里我们添加一个notify指令到安装任务。这将在任务运行后通知名为“Start Nginx”的处理程序。然后我们可以创建名为“Start Nginx”的处理程序。此处理程序是通知“Start Nginx”时调用的任务。 这个特定的处理程序使用服务模块，它可以启动，停止，重启，重新加载（等等）系统服务。在这种情况下，我们告诉ansible，我们要启动Nginx。 

+ 如果我已经安装了nginx，则安装nginx任务将不会运行，通知程序也将不会被调用。



##### 3.4 更多的任务（more tasks）

接下来，我们可以为此playbook添加更多的任务，并探索其他一些功能。

```yaml
- hosts: local
  connection: local
  become: yes
  become_user: root
  vars:
   - docroot: /var/www/serversforhackers.com/public
  tasks:
   - name: Add Nginx Repository
     apt_repository:
       repo: ppa:nginx/stable
       state: present
     register: ppastable

   - name: Install Nginx
     apt:
       pkg: nginx
       state: installed
       update_cache: true
     when: ppastable|success
     notify:
      - Start Nginx

   - name: Create Web Root
     file:
      path: '{{ docroot }}'
      mode: 775
      state: directory
      owner: www-data
      group: www-data
     notify:
      - Reload Nginx

  handlers:
   - name: Start Nginx
     service:
       name: nginx
       state: started

    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

现在有三个任务：

```bash
Add Nginx Repository # 使用apt_repository模块添加Nginx稳定PPA以获取最新的稳定版本的Nginx 。
Install Nginx 		   # 使用apt模块安装Nginx。
Create Web Root      # 最后创建一个Web根目录。
```

新的register和when指令，可以实现在某些事情发生后让ansible执行任务的功能。

您还可以注册模块操作的结果，并使用定义的变量根据注册（register）的变量值有条件（when）地执行操作。例如，注册通过shell模块运行命令的结果可以让您访问该命令的stdout。

同时还使用了一个变量, docroot变量在定义vars部分。然后将其用作创建定义目录的文件模块的目标参数。需要注意的是，path配置使用括号{{ var-name }}，这是Jinja2的模板。为了使ansible能够在括号内解析Jinja2模板变量，该行必须是单引号或双引号 - 例如，path: '{{ docroot }}' 而不是path: {{ docroot }}。不使用引号将导致错误。 这个playbook可以用通常的命令运行：

```
ansible-playbook -i ./hosts nginx.yml
```



### 4. ansible角色（roles）

角色才是ansible的精髓, 每个人可以做出自己的角色让别人使用, 也可以通过ansible-galaxy安装其他人的角色。

角色很适合组织多个相关任务并封装完成这些任务所需的数据。例如，安装nginx可能涉及添加软件包存储库，安装软件包和设置配置。 

此外，真实的配置通常需要额外的数据，如变量，文件，动态模板等等。这些工具可以与Playbook一起使用，但是我们可以通过将相关任务和数据组织成一个角色（role， 相关的结构）很快就能做得更好。 

角色有一个这样的目录结构：

```
roles
  rolename
   - files
   - handlers
   - meta
   - templates
   - tasks
   - vars
```

在每个子目录中（eg： files，handlers等等），ansible将自动搜索并读取叫做main.yml的yaml文件。 

接下来我们将分解nginx.yml文件内容为不同的组件，并将每个组件放在相应的目录中，以创建一个更干净，更完整的配置工具集。



##### 4.1 创建角色（creating a role）

我们可以使用ansible-galaxy命令来创建一个新角色。此工具可用于将角色保存到ansible的公共注册表，但是我通常只是使用它来在本地创建role的基础目录结构。

```bash
cd ~/ansible-example
mkdir roles
cd roles
ansible-galaxy init nginx
```

目录名称roles是一种惯例，在运行一个playbook时可以用来查找角色。该目录应该始终被命名roles，但并不强制。在roles目录中运行 ansible-galaxy init nginx 命令将创建新角色所需的目录和文件。

我们来看看我们新建的nginx角色的每个部分~/ansible-example/roles/nginx。

##### 4.2.1 文件（files）

files目录中没有main.yml文件.  首先，在files目录中，我们可以添加我们要复制到我们的服务器中的文件。对于nginx，我经常复制h5bp的nginx组件配置。我只需从github下载最新的信息，进行一些调整，并将它们放入files目录中。

```
~/ansible-example
 - roles
 - - nginx
 - - - files
 - - - - h5bp
```

我们稍后会看到，h5bp配置文件将通过复制模块添加到服务器。



##### 4.2.2 处理程序（handlers）

handlers约定必须包含main.yml文件。我们可以把曾经在nginx.yml 剧本中的定义的所有处理程序放入到handlers目录中。

handlers/main.yml 内容：

```yml
---
# handlers file for nginx

- name: Start Nginx
  service:
    name: nginx
    state: started

- name: Reload Nginx
  service:
    name: nginx
    state: reloaded
```

一旦handlers/main.yml中的处理程序定义好了，我们可以自由地从其他的yaml配置中引用它们。



##### 4.2.3 元（meta）

meta目录中约定必须包含main.yml文件。main.yml文件包含role元数据，包含的依赖关系。如果这个角色依赖于另一个角色，我们可以在这里定义。例如，nginx角色取决于安装ssl证书的ssl角色。 

meta/main.yml 内容：

```yml
---
dependencies:
  - { role: ssl }
```

如果我调用了“nginx”角色，它将尝试首先运行“ssl”角色。 否则我们可以省略此文件，或将角色定义为没有依赖关系：

```yml
---
dependencies: []
```



##### 4.2.4 模板（templates）

templates目录中没有main.yml文件，只包含.j2后缀的模板文件。 基于python的Jinja2模板引擎（和django的模板引擎很类似），模板文件可以包含模板变量。这里的文件应该以.j2为类型后缀（eg.uwsgi.j2），提倡但是不强制，也可以取其他的名字。

这是一个nginx服务器（“虚拟主机”）配置的例子。请注意，它使用了稍后在vars/main.yml文件中定义的一些变量。 我们的示例中的nginx配置文件位于templates/serversforhackers.com.conf.j2：

```nginx
server {
    # Enforce the use of HTTPS
    listen 80 default_server;
    server_name {{ domain }};
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl default_server;

    root /var/www/{{ domain }}/public;
    index index.html index.htm index.php;

    access_log /var/log/nginx/{{ domain }}.log;
    error_log  /var/log/nginx/{{ domain }}-error.log error;

    server_name {{ domain }};

    charset utf-8;

    include h5bp/basic.conf;

    ssl_certificate           {{ ssl_crt }};
    ssl_certificate_key       {{ ssl_key }};
    include h5bp/directive-only/ssl.conf;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt  { log_not_found off; access_log off; }

    location ~ \.php$ {
        include snippets/fastcgi.conf;
        fastcgi_pass unix:/var/run/php7.1-fpm.sock;
    }
}
```

这是一个相当标准的用于php应用程序的Nginx配置。这里有三个变量：`domain` `ssl_crt` `ssl_key` 这三个变量将在变量部分（vars）中定义。

##### 4.2.5 变量（vars）

vars目录包含一个main.yml文件, 在main.yml中我们可以列出将要使用的所有变量。 

vars/main.yml：

```yml
---
domain: serversforhackers.com
ssl_key: /etc/ssl/sfh/sfh.key
ssl_crt: /etc/ssl/sfh/sfh.crt
```

+ Note:如果您有敏感信息添加到变量文件中，则可以使用ansible-vault加密文件。



##### 4.2.6 任务（tasks）

tasks目录包含一个main.yml文件, 使用角色时运行的主文件是tasks/main.yml文件。看看我们的用例将会是什么样的：

```yml
---
- name: Add Nginx Repository
  apt_repository:
    repo: ppa:nginx/stable
    state: present

- name: Install Nginx
  apt:
    pkg: nginx
    state: installed
    update_cache: true
  notify:
    - Start Nginx # 在handler文件夹里

- name: Add H5BP Config
  copy:
    src: h5bp
    dest: /etc/nginx
    owner: root
    group: root

- name: Disable Default Site Configuration
  file:
    dest: /etc/nginx/sites-enabled/default
    state: absent

# `dest` in quotes as a variable is used!
- name: Add SFH Site Config
  register: sfhconfig
  template:
    src: serversforhackers.com.j2
    dest: '/etc/nginx/sites-available/{{ domain }}.conf' 
    owner: root
    group: root

# `src`/`dest` in quotes as a variable is used!
- name: Enable SFH Site Config
  file:
    src: '/etc/nginx/sites-available/{{ domain }}.conf'
    dest: '/etc/nginx/sites-enabled/{{ domain }}.conf'
    state: link

# `dest` in quotes as a variable is used!
- name: Create Web root
  file:
    dest: '/var/www/{{ domain }}/public'
    mode: 775
    state: directory
    owner: www-data
    group: www-data
  notify:
    - Reload Nginx

# `dest` in quotes as a variable is used!
- name: Web Root Permissions
  file:
   dest: '/var/www/{{ domain }}'
   mode: 775
   state: directory
   owner: www-data
   group: www-data
   recurse: yes
  notify:
    - Reload Nginx
```

这一系列任务使得nginx能被完整的安装。任务按照出现的顺序完成以下工作：

```
1 添加nginx / stable库
2 安装并启动nginx
3 添加H5BP配置文件
4 从sites-enabled目录中删除文件的符号链接来禁用默认的nginx配置
5 将serversforhackers.com.conf.j2虚拟主机模板复制到nginx配置中，渲染模板
6 通过将其符号链接到sites-enabled目录来启用Nginx服务器配置
7 创建Web根目录
8 更改项目根目录的权限（递归），该目录位于之前创建的Web根目录之上
```

有一些新的模块（和一些我们已经涵盖的新用途），包括复制，模板和文件模块。通过设置每个模块的参数，我们可以做一些有趣的事情，例如确保文件“不存在”（如果存在则删除它们）的state: absent，或者通过创建一个文件作为符号链接的state: link。您应该检查每个模块的文档，以查看可以用它们完成哪些有趣和有用的事情。

参加官方文档: https://docs.ansible.com/ansible/latest/modules/



##### 4.3 运行角色（running the Role）

要对服务器运行一个或多个角色，我们将重新使用另一个playbook。该playbook与roles目录位于同一个目录中，同一层级。当我们用ansible-playbook命令运行的时候需要先cd进入到该目录中。 

让我们创建一个“主”的yaml文件（被ansible-playbook命令执行的文件），该文件定义要使用的角色以及运行它们的主机： 文件~/ansible-example/server.yml位于与roles目录相同的目录中：(注意是和roles目录同层级!!!!!)

```yml
---
- hosts: local
  connection: local
  roles:
    - nginx # 这个是你刚才写的nginx role
```

所以，我们只是定义角色，而不是在本playbook文件中定义所有的变量和任务。角色负责具体细节。然后我们可以运行角色：

```bash
ansible-playbook -i ./hosts server.yml
```

以下是运行nginx角色的playbook文件的输出：

```bash
PLAY [all] ********************************************************************

GATHERING FACTS ***************************************************************
ok: [127.0.0.1]

TASK: [nginx | Add Nginx Repository] ******************************************
changed: [127.0.0.1]

TASK: [nginx | Install Nginx] *************************************************
changed: [127.0.0.1]

TASK: [nginx | Add H5BP Config] ***********************************************
changed: [127.0.0.1]

TASK: [nginx | Disable Default Site] ******************************************
changed: [127.0.0.1]

TASK: [nginx | Add SFH Site Config] *******************************************
changed: [127.0.0.1]

TASK: [nginx | Enable SFH Site Config] ****************************************
changed: [127.0.0.1]

TASK: [nginx | Create Web root] ***********************************************
changed: [127.0.0.1]

TASK: [nginx | Web Root Permissions] ******************************************
ok: [127.0.0.1]

NOTIFIED: [nginx | Start Nginx] ***********************************************
ok: [127.0.0.1]

NOTIFIED: [nginx | Reload Nginx] **********************************************
changed: [127.0.0.1]

PLAY RECAP ********************************************************************
127.0.0.1                  : ok=8   changed=7   unreachable=0    failed=0
```

我们将所有各种组件放在一起，形成一致的角色，现在已经安装并配置了nginx！



### 5. ansible事实(facts)

请注意，运行剧本时的第一行总是“收集事实”。 在运行任何任务之前，ansible将收集有关其配置的系统的信息。这些被称为事实，并且包括广泛的系统信息，如CPU核心数量，可用的ipv4和ipv6网络，挂载的磁盘，Linux发行版等等。

事实在“任务”或“模板”配置中通常很有用。例如，nginx通常设置为使用与cpu内核一样多的工作处理器。知道这一点，您可以选择如下设置nginx.conf.j2文件的模板：

```nginx
user www-data;
worker_processes {{ ansible_processor_cores }};
pid /var/run/nginx.pid;

# And other configurations...
```

或者如果你具有多个cpu的服务器，则可以使用：

```nginx
user www-data;
worker_processes {{ ansible_processor_cores * ansible_processor_count }};
pid /var/run/nginx.pid;

# And other configurations...
```

所有的ansible facts全局变量都是以“anisble_”为前缀，并且可以在其他任何地方使用。 尝试对你的本地机器运行以下内容以查看可用的事实：

```bash
# Run against a local server
# Note that we say to use "localhost" instead of defining a hosts file here!
ansible -m setup --connection=local localhost

# Run against a remote server
ansible -i ./hosts remote -m setup
```



### 6. ansible遇到的问题总结

+ 遇到角色没有的问题, ERROR! the role 'Stouts.ntp' was not found

  通过ansible-galaxy安装 

  ```
  ansible-galaxy install stouts.ntp
  ```

+ ansible的全局配置ansible.cfg

  如默认是否需要输入密码、是否开启sudo认证、action_plugins插件的位置、hosts主机组的位置、是否开启log功能、默认端口、key文件位置等等。
  
  参见: https://www.cnblogs.com/paul8339/p/6159220.htm

+ host文件把一个组作为另一个组的子成员 

  ```
  [atlanta]
  host1
  host2
  
  [raleigh]
  host2
  host3
  
  [southeast:children]
  atlanta
  raleigh
  
  [southeast:vars]
  some_server=foo.southeast.example.com
  halon_system_timeout=30
  self_destruct_countdown=60
  escape_pods=2
  
  [usa:children]
  southeast
  northeast
  southwest
  northwest
  ```

  atlanta raleigh将作为southeast子组，继承父组southeast中some_server等变量

  

+ 限制脚本只在指定的ip对应的机器上执行。

   -l <SUBSET>, --limit <SUBSET>
   
	```bash
   ansible-playbook myplaybook.yml -l 10.11.12.13
   ```



+ 打标签归类, 指定归类相关的task

  ```yml
  ---
  - include: restart.yml
    tags:
      - tag2
  - include: push.yml
    tags:
      - tag1
  ```

  tags主要目的是单独执行指定的tag，使用-t 或者--tags 表示。旧版本不是这样写的，会直接在include后面加上tags=xxx,但是在新版本的ansible执行时虽然不报错，但是也不执行该tag。



### 7. 参考资料

+ [非常好的Ansible入门教程（超简单）](https://blog.csdn.net/pushiqiang/article/details/78126063)

+ [Ansible 快速入门](https://www.cnblogs.com/dachenzi/p/8916521.html)

+ [ansible自动化运维教程](https://www.w3cschool.cn/automate_with_ansible/ )

+ https://docs.ansible.com/ansible/latest/modules/ 官方文档 

  

