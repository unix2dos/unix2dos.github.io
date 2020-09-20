---
title: hexo搭建博客
urlname: hexo_blog
tag: hexo
abbrlink: b37651
categories: []
date: 2016-12-01 17:54:46
---

# 1. 安装hexo

+ 安装node.js
+ 安装hexo

	```
	npm install -g hexo-cli
	mkdir hexo
	hexo init hexo
	cd hexo
	```

<!-- more -->

# 2. hexo配置

+ 因为主站有个配置, 主题也有个配置, 建议两个配置合并一起, 需要Hexo版本在 3 以上

+ 在站点的 `source/_data` 目录下新建 `next.yml` 文件（`_data`目录可能需要新建）迁移站点配置文件和主题配置文件中的配置到 `next.yml` 中(包含了`_config.yml`和`theme.yml`)
  
	```
	hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
	```
	
+ 不渲染 README

  将skip_render参数的值设置上。skip_render: README.md
  使用hexo d 命令就不会在渲染 README.md 这个文件了。



# 3. 绑定域名

+ 为自己的 github 生成一个公钥私钥对

+ 建立带用户名的仓库 unix2dos.github.io

- CNAME 放到 source 文件夹, 里面写上 www.liuvv.com
- 向你的 DNS 配置中添加 3 条记录

```
@          A             192.30.252.153
@          A             192.30.252.154
www      CNAME           unix2dos.github.io.
```



# 4. hexo插件

```shell
npm install hexo --save
npm install hexo-deployer-git --save

npm ls --depth 0  //查看丢失的包
npm install hexo-generator-archive --save //逐一安装缺失的包


### hexo-next 主题

# 第一台电脑
cd themes
git submodule add https://github.com/unix2dos/hexo-theme-next next
cd next

# 第二台电脑
git submodule update --init
cd themes/next

# 分享按钮
git clone https://github.com/theme-next/theme-next-needmoreshare2 source/lib/needsharebutton  

# 丝带
git clone https://github.com/theme-next/theme-next-canvas-ribbon source/lib/canvas-ribbon

# 蜘蛛网
git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest

# 三种特效
git clone https://github.com/theme-next/theme-next-three source/lib/three 

# 特殊汉字
git clone https://github.com/theme-next/theme-next-han source/lib/Han

# 快速点击
git clone https://github.com/theme-next/theme-next-fastclick source/lib/fastclick

# 懒加载
git clone https://github.com/theme-next/theme-next-jquery-lazyload source/lib/jquery_lazyload

# 顶部的进度
git clone https://github.com/theme-next/theme-next-pace source/lib/pace 

# 图片展示
git clone https://github.com/theme-next/theme-next-fancybox3 source/lib/fancybox 

# 文字显示加空格
git clone https://github.com/theme-next/theme-next-pangu.git source/lib/pangu

# 读取进度
git clone https://github.com/theme-next/theme-next-reading-progress source/lib/reading_progress 


### hexo-next 插件

1. npm install hexo-symbols-count-time --save   # 统计字数

symbols_count_time:
  symbols: true
  time: true
  total_symbols: true
  total_time: true
  separated_meta: true
  item_text_post: true
  item_text_total: false
  awl: 4
  wpm: 275


2. npm install hexo-abbrlink --save # 链接持久

permalink: post/:abbrlink.html
abbrlink:
  alg: crc16 #support crc16(default) and crc32
  rep: hex    #support dec(default) and hex
  
  
3. npm install hexo-auto-category --save #自动分类

auto_category:
 enable: true
 depth:
 
 
4. npm install hexo-generator-searchdb --save # 本地搜索

search:
  path: search.json
  field: post
  format: html
  limit: 10000
  content: true
  
#然后打开本地local_search
local_search:
	enable: true
  

5.  gittalk评论系统(禁止使用, 建议使用 disqus)

https://github.com/theme-next/hexo-theme-next/pull/464
https://asdfv1929.github.io/2018/01/20/gitalk/
https://github.com/settings/developers

Homepage URL 和 Authorization callback URL 都填写自己配置的域名
```



# 5. 问题解决方案

### 5.1 生成空白页

+ 生成页面如果空白的话, 换个主题再重新生成一次
+ 更新下主题仓库

### 5.2  WARN No layout

看看主题里面究竟有没有东西,文件夹名字和主题是否对应

### 5.3 使用链接持久后图片无法显示

+ https://github.com/rozbo/hexo-abbrlink/issues/19

```javascript
# vi node_modules/hexo-asset-image/index.js     #24行

// var endPos = link.length-1; // 换成下面的这句话
var endPos = link.length-5; //因为我的permalink: p/:abbrlink.html,  这里要改成-5
```

### 5.4 证书更新

1. coding

+ 暂停dns解析github 
+ coding申请证书
+ 再打开解析github

2. github

### 5.5 主题更新

+ fork 到自己github, 用新的分支, 修改了一些language
+ 定期同步最新的仓库主题



# 6. 常用命令

```html
hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml

---
title: ""
date: 2018-05-19 17:54:46
tags:
- golang
- linux
---

<!-- more -->


![1](Kademlia_DHT_KRPC_BitTorrent协议/1.png)
```

