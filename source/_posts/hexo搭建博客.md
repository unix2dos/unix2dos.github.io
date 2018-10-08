---
title: 'hexo搭建博客'
date: 2016-12-01 17:54:46
tag: hexo
---

### 安装hexo

+ 安装node.js
+ 安装hexo

	```
	npm install -g hexo-cli
	mkdir hexo
	hexo init hexo
	cd hexo
	```

<!-- more -->

### hexo配置

+ 因为主站有个配置, 主题也有个配置, 建议两个配置合并一起, 需要Hexo版本在 3 以上

+ 在站点的 source/_data 目录下新建 next.yml 文件（_data目录可能需要新建）迁移站点配置文件和主题配置文件中的配置到 next.yml 中
  
	```
	hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
	```
	
+ 不渲染 README

  将skip_render参数的值设置上。skip_render: README.md
  使用hexo d 命令就不会在渲染 README.md 这个文件了。



### github pages

+ 建立带用户名的仓库 unix2dos.github.io



### 第三方插件

+ algolia //放弃采用local-search

```
npm install --save hexo-algolia
export HEXO_ALGOLIA_INDEXING_KEY=03a0650ecd0ead0dfd9031178da3f591 (admin key)
hexo algolia  --config source/_data/next.yml 
```



### 另外一台电脑安装hexo

```
npm install hexo --save
npm install hexo-deployer-git --save
```



### 绑定域名

+ CNAME 放到 source 文件夹, 里面写上 www.xuanyueting.com
+ 向你的 DNS 配置中添加 3 条记录

```
@          A             192.30.252.153
@          A             192.30.252.154
www      CNAME           username.github.io.
```



### 常用命令

```
hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
hexo algolia  --config source/_data/next.yml 


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



### 问题解决方案

1. 生成页面如果空白的话, 换个主题再来一次

2. local search 一直 loading 看错误信息, 解决非 utf-8问题

3. 不生成页面的话: 

   ```
   npm ls --depth 0  //查看丢失的包
   npm install hexo-generator-archive --save //逐一安装缺失的包
   ```

5. WARN  No layout

   看看主题里面究竟有没有东西,文件夹名字和主题是否对应