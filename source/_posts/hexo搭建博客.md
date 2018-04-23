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
	
### github pages

+ 建立带用户名的仓库 unix2dos.github.io

### 第三方插件

algolia

```
npm install --save hexo-algolia
export HEXO_ALGOLIA_INDEXING_KEY=03a0650ecd0ead0dfd9031178da3f591 (admin key)
hexo algolia  --config source/_data/next.yml 
```


### 另外一台安装
	
```
npm install hexo --save
npm install hexo-deployer-git --save
```

### 问题

+ 生成页面如果空白的话, 换个主题再来一次

