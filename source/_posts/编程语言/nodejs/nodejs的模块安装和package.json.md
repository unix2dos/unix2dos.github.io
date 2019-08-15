---
title: nodejs的模块安装和package.json
tags:
  - nodejs
abbrlink: d100a0c2
categories:
  - 编程语言
  - nodejs
date: 2019-05-10 20:01:00
---



### 1. npm 介绍

[npm](https://www.npmjs.com/package/npm) 是 Node 的模块管理器，功能极其强大。它是 Node 获得成功的重要原因之一。

npm不需要单独安装。在安装Node的时候，会连带一起安装npm。但是，Node附带的npm可能不是最新版本，最好用下面的命令，更新到最新版本。

```shell
npm install npm@latest -g 
```

上面的命令中，@latest表示最新版本，-g表示全局安装。所以，命令的主干是npm install npm，也就是使用npm安装自己。之所以可以这样，是因为npm本身与Node的其他模块没有区别。

<!-- more -->



### 2. npm 安装

#### 2.1 npm install 

[npm install](https://docs.npmjs.com/cli/install) 命令用来安装模块到node_modules目录。

```shell
 $ npm install <packageName> 
```



安装之前，npm install会先检查，node_modules目录之中是否已经存在指定模块。如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也是如此。

如果你希望，一个模块不管是否安装过，npm 都要强制重新安装，可以使用-f或--force参数。

```shell
 $ npm install <packageName> --force
```



#### 2.2 npm update

如果想更新已安装模块，就要用到[npm update](https://docs.npmjs.com/cli/update)命令。

```shell
 $ npm update <packageName> 
```

它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。



#### 2.3 registry

npm update命令怎么知道每个模块的最新版本呢？

答案是 npm 模块仓库提供了一个查询服务，叫做 registry 。以 npmjs.org 为例，它的查询服务网址是 https://registry.npmjs.org/ 。

这个网址后面跟上模块名，就会得到一个 JSON 对象，里面是该模块所有版本的信息。比如，访问 <https://registry.npmjs.org/react>，就会看到 react 模块所有版本的信息。

它跟下面命令的效果是一样的。

```shell
 $ npm view react  
```



registry 网址的模块名后面，还可以跟上版本号或者标签，用来查询某个具体版本的信息。比如， 访问 https://registry.npmjs.org/react/v0.14.6 ，就可以看到 React 的 0.14.6 版。

返回的 JSON 对象里面，有一个dist.tarball属性，是该版本压缩包的网址。

```json
dist: {   
	shasum: '2a57c2cf8747b483759ad8de0fa47fb0c5cf5c6a',   
	tarball: 'http://registry.npmjs.org/react/-/react-0.14.6.tgz'  
},
```



到这个网址下载压缩包，在本地解压，就得到了模块的源码。`npm install`和`npm update`命令，都是通过这种方式安装模块的。



#### 2.4 缓存目录

npm install或npm update命令，从 registry 下载压缩包之后，都存放在本地的缓存目录。

这个缓存目录，在 Linux 或 Mac 默认是用户主目录下的.npm目录，在 Windows 默认是%AppData%/npm-cache。通过配置命令，可以查看这个目录的具体位置。

```shell
 $ npm config get cache 
 /Users/liuwei/.npm
```

 

.npm目录保存着大量文件，清空它的命令如下。 

```shell
$ rm -rf ~/.npm/ 
或
$ npm cache clean 
```



#### 2.5 模块的安装过程

总结一下，Node模块的安装过程是这样的。

1. 发出npm install命令
2. npm 向 registry 查询模块压缩包的网址
3. 下载压缩包，存放在~/.npm目录
4. 解压压缩包到当前项目的node_modules目录



注意，一个模块安装以后，本地其实保存了两份。一份是~/.npm目录下的压缩包，另一份是node_modules目录下解压后的代码。

但是，运行npm install的时候，只会检查node_modules目录，而不会检查~/.npm目录。也就是说，如果一个模块在～/.npm下有压缩包，但是没有安装在node_modules目录中，npm 依然会从远程仓库下载一次新的压缩包。



### 3. package.json

管理本地安装 npm 包的最好方式就是创建 package.json 文件。一个 package.json 文件可以有以下几点作用：

+ 作为一个描述文件，描述了你的项目依赖哪些包

+ 允许我们使用 “语义化版本规则”（后面介绍）指明你项目依赖包的版本

+ 让你的构建更好地与其他开发者分享，便于重复使用

  

使用 `npm init` 即可在当前目录创建一个 `package.json` 文件。如果嫌回答这一大堆问题麻烦，可以直接输入` npm init -—yes` 跳过回答问题步骤，直接生成默认值的 package.json 文件：

```json
{
  "name": "package",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```



#### 3.1 package.json 的内容



##### 3.1.1 `package.json` 文件至少要有两部分内容：

1. “name”
   - 全部小写，没有空格，可以使用下划线或者横线
2. “version” 
   - x.x.x 的格式
   - 符合“语义化版本规则”



##### 3.1.2 其他内容：

+ description：描述信息，有助于搜索
+ main: `入口文件`，一般都是 index.js. 当使用require()语法来加载一个模块时，就会看此值
+ scripts：支持的脚本，默认是一个空的 test
+ keywords：关键字，有助于在人们使用 npm search 搜索时发现你的项目
+ author：作者信息
+ license：默认是 MIT
+ bugs：当前项目的一些错误信息，如果有的话



##### 3.1.3 scripts属性

可以指定npm命令缩写。

```
  "scripts": {
    "start": "node store.js"
  },
```



执行npm run start仍然可以运行成功，通过scripts属性npm run start等价于node store.js。关于scripts的更具体的使用[请看这里](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)。



#### 3.2 指定依赖的包

我们需要在 `package.json` 文件中指定项目依赖的包，这样别人在拿到这个项目时才可以使用 `npm install` 下载。

包有两种依赖方式：

1. `dependencies`：在生产环境中需要用到的依赖
2. `devDependencies`：在开发、测试环境中用到的依赖



举个例子：

```
{
    "name": "my-weex-demo",
    "version": "1.0.0",
    "description": "a weex project",
    "main": "index.js",
    "devDependencies": {
        "babel-core": "^6.14.0",
        "babel-loader": "^6.2.5",
        "babel-preset-es2015": "^6.18.0",
        "vue-loader": "^10.0.2",
        "eslint": "^3.5.0",
        "serve": "^1.4.0",
        "webpack": "^1.13.2",
        "weex-loader": "^0.3.3",
        "weex-builder": "^0.2.6"
    },
    "dependencies": {
        "weex-html5": "^0.3.2",
        "weex-components": "*"
    }
}
```



#### 3.3 semantic versioning（语义化版本规则）



dependencies 的内容，以 "weex-html5": "^0.3.2" 为例，我们知道 key 是依赖的包名称，value 是这个包的版本。那版本前面的 ^ 或者版本直接是一个 * 是什么意思呢？这就是 npm 的 “Semantic versioning”，简称”Semver”，中文含义即“语义化版本规则”。

在开发中我们有过这样的经验：有时候依赖的包升级后大改版，之前提供的接口不见了，这对使用者的项目可能造成极大的影响。因此我们在声明对某个包的依赖时需要指明是否允许 update 到新版本，什么情况下允许更新。这就需要先了解 npm 包提供者应该注意的版本号规范。



如果一个项目打算与别人分享，应该从 1.0.0 版本开始。以后要升级版本应该遵循以下标准：

```
补丁版本：解决了 Bug 或者一些较小的更改，增加最后一位数字，比如 1.0.1
小版本：增加了新特性，同时不会影响之前的版本，增加中间一位数字，比如 1.1.0
大版本：大改版，无法兼容之前的，增加第一位数字，比如 2.0.0

了解了提供者的版本规范后， npm 包使用者就可以针对自己的需要填写依赖包的版本规则。
```



作为使用者，我们可以在 package.json 文件中写明我们可以接受这个包的更新程度（假设当前依赖的是 1.0.4 版本）：

如果只打算接受补丁版本的更新（也就是最后一位的改变），就可以这么写： 

```
1.0
1.0.x
~1.0.4
```

如果接受小版本的更新（第二位的改变），就可以这么写： 

```
1
1.x
^1.0.4
```

如果可以接受大版本的更新（自然接受小版本和补丁版本的改变），就可以这么写： 

```
*
x
```


小结一下：总共三种版本变化类型，接受依赖包哪种类型的更新，就把版本号准确写到前一位。





### 4. 其他安装知识



#### 4.1 安装参数 --save 和 --save -dev

添加依赖时我们可以手动修改 package.json 文件，添加或者修改 dependencies devDependencies 中的内容即可。另一种更酷的方式是用命令行，在使用 npm install 时增加 --save 或者 --save -dev 后缀：

```
npm install <package_name> --save 表示将这个包名及对应的版本添加到 package.json的 dependencies

npm install <package_name> --save-dev 表示将这个包名及对应的版本添加到 package.json的 devDependencies
```

dependencies：在生产环境中需要用到的依赖

devDependencies：在开发、测试环境中用到的依赖



#### 4.2 全局安装 package

如果你想要直接在命令行中使用某个包，比如 jshint ，你可以全局安装它。全局安装比本地安装多了个 -g:

```
npm install -g <package-name>
```

安装后可以使用 npm ls -g --depth=0 查看安装在全局第一层的包。

##### 4.2.1 全局安装的权限问题

在全局安装时可能会遇到 EACCES 权限问题, 可以如下解决：

1.使用 sudo 简单粗暴，但是治标不治本

2.修改 npm 全局默认目录的权限

先获取 npm 全局目录：`npm config get prefix`，一般都是 /usr/local； 然后修改这个目录权限为当前用户：

```shell
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
```



#### 4.3 package-lock.json

原来package.json文件只能锁定大版本，也就是版本号的第一位，并不能锁定后面的小版本，你每次npm install都是拉取的该大版本下的最新的版本，为了稳定性考虑我们几乎是不敢随意升级依赖包的，这将导致多出来很多工作量，测试/适配等，所以package-lock.json文件出来了，当你每次安装一个依赖的时候就锁定在你安装的这个版本。

那如果我们安装时的包有bug，后面需要更新怎么办？

 在以前可能就是直接改package.json里面的版本，然后再npm install了，但是5版本后就不支持这样做了，因为版本已经锁定在package-lock.json里了，所以我们只能npm install xxx@x.x.x  这样去更新我们的依赖，然后package-lock.json也能随之更新。





### 参考资料:

http://www.ruanyifeng.com/blog/2016/01/npm-install.html

http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html

https://blog.csdn.net/u011240877/article/details/76582670