---
title: hugo搭建博客
tags:
  - hugo
categories:
  - 5-随笔
  - 个人记录
abbrlink: 74f76184
date: 2021-01-30 00:00:00
---

博客之前是用 hexo 来搭建的, 问为什么要转移到 hugo, 就是一个字: 太慢.

但是除了快,  hexo 好多牛逼的插件, hugo 目前还没有, 然后模板也比较丑.

<!-- more -->

# 1. 安装和配置

### 1.1 安装Hugo

```bash
# 安装 hugo
brew install hugo

# 创建项目
hugo new site hugo-demo && cd hugo-demo 

# 设置主题
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/
echo 'theme = "ananke"' >> config.toml 

# 新建文章
hugo new posts/my-first-post.md

# 启动 server 预览
hugo server
```

在浏览器输入 `http://localhost:1313` 即可查看效果

使用如下代码部署编译完成的静态页面文件：

```bash
hugo -D
```

### 1.2 腾讯云静态部署

```bash
npm install -g @cloudbase/cli
tcb login
cloudbase hosting deploy ./public  -e EnvID -r bj # 此处的 EnvID 替换为腾讯云CloudBase环境 ID, -r bj 是北京
```



# 2. 主题even

```bash
git submodule add https://github.com/olOwOlo/hugo-theme-even.git themes/even


# Take a look inside the exampleSite folder of this theme. You'll find a file called config.toml. To use it, copy the config.toml in the root folder of your Hugo site. Feel free to change it.
cp themes/even/exampleSite/config.toml ./


#  For this theme, you should use post instead of posts, namely hugo new post/some-content.md
mv  content/posts/   content/post/
```



### 2.1 图片相对路径

config.toml 设置

```ini
uglyurls = true
```

如果不行, 再设置环境变量

```bash
export HUGO_UGLYURLS=true
```

### 2.2 评论

config.toml 设置 valine 的 id

```ini
params.valine
```

### 2.3 唯一地址

TODO: 没有找到`hexo-abbrlink`类似的插件, 为了和以前的地址兼容, 还是一件麻烦的事情.

### 2.4 本地搜索

TODO: 没找到简单的方案



# 3. 错误

### 3.1  tags 语法问题

executing "_internal/schema.html" at <.Params.tags>: range can't iterate over mongodb

tags: mongodb  这样就会导致 `tags` 不能迭代，需要改成 `tags: [mongodb]` 才能解决这个 Bug。




# 4. 参考资料

+ https://cloud.tencent.com/document/product/1210/43389
+ https://blog.lxdlam.com/post/9cc3283b/

