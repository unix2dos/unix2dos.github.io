# 1. 安装



```bash
brew install hugo


```



```
hugo new site hugo-demo && cd hugo-demo
```





```plaintext


git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/
echo 'theme = "ananke"' >> config.toml

```



```
hugo new posts/my-first-post.md
hugo server
```



打开配置文件 `config.toml`，文件示例如下：

```bash
baseURL = "http://example.org/" #“baseURL” 为默认或者自定义域名。
languageCode = "en-us"
title = "My New Hugo Site"  #修改 “title” 的值为网站名称。
theme = "ananke"
```





在浏览器输入 `http://localhost:1313` 即可查看效果





使用如下代码部署编译完成的静态页面文件：

```bash
hugo -D
```





```
cloudbase hosting deploy ./public  -e EnvID
```





\+ 评论

\+ 唯一地址

\+ 主题

```bash
git clone https://github.com/olOwOlo/hugo-theme-even themes/even


```









参考资料

https://gohugo.io/getting-started/quick-start/

https://cloud.tencent.com/document/product/1210/43389

https://scarletsky.github.io/2019/05/02/migrate-hexo-to-hugo/