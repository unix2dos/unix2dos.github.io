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



```bash
npm install -g @cloudbase/cli
tcb login
cloudbase hosting deploy ./public  -e EnvID  #此处的 EnvID 替换为腾讯云CloudBase环境 ID
```



hugo-3gejy0jl4bd348e7





# 2. 主题

```bash
git clone https://github.com/olOwOlo/hugo-theme-even themes/even

# Take a look inside the exampleSite folder of this theme. You'll find a file called config.toml. To use it, copy the config.toml in the root folder of your Hugo site. Feel free to change it.
cp themes/even/exampleSite/config.toml ./


#  For this theme, you should use post instead of posts, namely hugo new post/some-content.md
mv  content/posts/   content/post/
```



# 3. 插件

### 3.1 评论



### 3.2 唯一地址



# 4. 错误

### 4.1  Front-matter 

executing "_internal/schema.html" at <.Params.tags>: range can't iterate over mongodb

tags: mongodb  这样就会导致 `tags` 不能迭代，需要改成 `tags: [mongodb]` 才能解决这个 Bug。





# 4. 参考资料

+ https://gohugo.io/getting-started/quick-start/
+ https://cloud.tencent.com/document/product/1210/43389
+ https://scarletsky.github.io/2019/05/02/migrate-hexo-to-hugo/

