---
title: jekyll搭建博客
tags:
  - jekyll
categories:
  - 5-随笔
  - 个人记录
abbrlink: 139d19f4
date: 2021-02-02 00:00:00
---

# 1. 安装使用

```bash
sudo gem install jekyll
```

本来以为`jekyll`是最简单部署的, 实践发现, 一点也没少折腾. 

<!-- more -->

### 1.1 报错

+ requires ruby version >= 2.4.0.

  ```bash
  brew reinstall ruby
  echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc
  ```

+ Jekyll - command not found

  ```
  sudo gem install -n /usr/local/bin jekyll
  ```

  

### 1.2 新建 blog

```bash
jekyll new myblog
cd myblog
jekyll serve # 启动 server
```



# 2. 使用

### 2.1 简单修改主页

在 github 创建好 githubpage

```bash
git clone git@github.com:unix2dos/httprun.git
git checkout gh-pages
```


修改 index.md 即可



# 3. 参考资料

+ https://jekyllcn.com/docs/quickstart/