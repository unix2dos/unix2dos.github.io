---
title: hexo搭建博客
urlname: hexo_blog
tag: hexo
abbrlink: b37651
categories: []
date: 2016-12-01 17:54:46
---

### 1. 安装hexo

+ 安装node.js
+ 安装hexo

	```
	npm install -g hexo-cli
	mkdir hexo
	hexo init hexo
	cd hexo
	```

<!-- more -->

### 2. hexo配置

+ 因为主站有个配置, 主题也有个配置, 建议两个配置合并一起, 需要Hexo版本在 3 以上

+ 在站点的 `source/_data` 目录下新建 `next.yml` 文件（`_data`目录可能需要新建）迁移站点配置文件和主题配置文件中的配置到 `next.yml` 中(包含了`_config.yml`和`theme.yml`)
  
	```
	hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
	```
	
+ 不渲染 README

  将skip_render参数的值设置上。skip_render: README.md
  使用hexo d 命令就不会在渲染 README.md 这个文件了。



### 3. github pages + 绑定域名

+ 建立带用户名的仓库 unix2dos.github.io

- CNAME 放到 source 文件夹, 里面写上 www.liuvv.com
- 向你的 DNS 配置中添加 3 条记录

```
@          A             192.30.252.153
@          A             192.30.252.154
www      CNAME           unix2dos.github.io.
```



### 4. 常用命令

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



### 5. 另外一台电脑同步hexo

```shell
npm install hexo --save
npm install hexo-deployer-git --save

npm ls --depth 0  //查看丢失的包
npm install hexo-generator-archive --save //逐一安装缺失的包


### hexo-next 主题
git clone https://github.com/theme-next/hexo-theme-next themes/next //下载next主题

cd themes/next

git clone https://github.com/theme-next/theme-next-needmoreshare2 source/lib/needsharebutton  //分享按钮

git clone https://github.com/theme-next/theme-next-canvas-ribbon source/lib/canvas-ribbon //丝带

git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest //蜘蛛网

git clone https://github.com/theme-next/theme-next-three source/lib/three // 三种特效

git clone https://github.com/theme-next/theme-next-han source/lib/Han //特殊汉字

git clone https://github.com/theme-next/theme-next-fastclick source/lib/fastclick //快速点击

git clone https://github.com/theme-next/theme-next-jquery-lazyload source/lib/jquery_lazyload //懒加载

git clone https://github.com/theme-next/theme-next-pace source/lib/pace //顶部的进度

git clone https://github.com/theme-next/theme-next-fancybox3 source/lib/fancybox //图片展示

git clone https://github.com/theme-next/theme-next-pangu.git source/lib/pangu //文字显示加空格

git clone https://github.com/theme-next/theme-next-reading-progress source/lib/reading_progress //读取进度


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
 
 
4. npm install hexo-generator-searchdb --save # 本地搜索,next主题把本地搜索打开
local_search:
	enable: true
  
5. 归档->时间轴,  搜索->站内搜索
```





### 6. 问题解决方案

1. 生成页面如果空白的话, 换个主题再重新生成一次

2. WARN  No layout

   看看主题里面究竟有没有东西,文件夹名字和主题是否对应
   
3. 使用链接持久后图片无法显示(https://github.com/rozbo/hexo-abbrlink/issues/19)

   ```javascript
       if(/.*\/index\.html$/.test(link)) {
         // when permalink is end with index.html, for example 2019/02/20/xxtitle/index.html
         // image in xxtitle/ will go to xxtitle/index/
         appendLink = 'index/';
         var endPos = link.lastIndexOf('/');
       }
    else {
         // var endPos = link.length-1; // 换成下面的这句话
         var endPos = link.length-5; //因为我的permalink: p/:abbrlink.html,  这里要改成-5
       }
   ```
   



### 7. 参考资料

+ https://iuok.me/posts/3159684541/ 插件