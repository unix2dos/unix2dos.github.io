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
cloudbase hosting deploy ./public  -e EnvID -r bj #此处的 EnvID 替换为腾讯云CloudBase环境 ID

```



# 2. 主题

```bash
git clone https://github.com/olOwOlo/hugo-theme-even themes/even

# Take a look inside the exampleSite folder of this theme. You'll find a file called config.toml. To use it, copy the config.toml in the root folder of your Hugo site. Feel free to change it.
cp themes/even/exampleSite/config.toml ./


#  For this theme, you should use post instead of posts, namely hugo new post/some-content.md
mv  content/posts/   content/post/
```



### 2.1 图片显示



# 3. 插件

### 3.1 评论

params.valine

### 3.2 唯一地址

原本的Hexo博客使用了`hexo-abbrlink`插件，目的是为每篇文章生成由字母和数字组成的随机URL，这样有利于SEO。迁移到Hugo后没找到类似的插件，只能用自带的`slug`功能来代替。



```fallback
---
abbrlink: 71bd19d3
slug: 71bd19d3
---
```







修改 `archetypes/default.md` 添加如下一行：

```yaml
---
#...
slug: {{ substr (md5 (printf "%s%s" .Date (replace .TranslationBaseName "-" " " | title))) 4 8 }}
#...
---
```

这样在每次使用 `hugo new` 的时候就会自动填写一个永久链接了。



```fallback
[permalinks]
  posts = "/posts/:slug.html"
  
enablePermalinks = true
```











# 4. 错误

### 4.1  Front-matter 

executing "_internal/schema.html" at <.Params.tags>: range can't iterate over mongodb

tags: mongodb  这样就会导致 `tags` 不能迭代，需要改成 `tags: [mongodb]` 才能解决这个 Bug。





# 5. 参考资料

+ https://gohugo.io/getting-started/quick-start/
+ https://cloud.tencent.com/document/product/1210/43389
+ https://scarletsky.github.io/2019/05/02/migrate-hexo-to-hugo/
+ https://lewky.cn/posts/hugo-4.html/
+ https://blog.lxdlam.com/post/9cc3283b/

