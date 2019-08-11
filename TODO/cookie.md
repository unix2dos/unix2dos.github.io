1

Cookie 是一些数据, 存储于你电脑上的文本文件中。

当 web 服务器向浏览器发送 web 页面时，在连接关闭后，服务端不会记录用户的信息。

Cookie 的作用就是用于解决 "如何记录客户端的用户信息":

- 当用户访问 web 页面时，他的名字可以记录在 cookie 中。
- 在用户下一次访问该页面时，可以在 cookie 中读取用户访问记录。





Web 服务器通过发送一个称为 Set-Cookie 的 HTTP 消息头来创建一个 cookie，Set-Cookie消息头是一个字符串，其格式如下（中括号中的部分是可选的）：

Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]





expires**选项 (***MaxAge***选项)**



紧跟 cookie 值后面的每个选项都以分号和空格分开，每个选择都指定了 cookie 在什么情况下应该被发送至服务器。第一个选项是过期时间（expires），指定了 cookie 何时不会再被发送至服务器，随后浏览器将删除该 cookie。该选项的值是一个 Wdy, DD-Mon-YYYY HH:MM:SS GMT 日期格式的值，例如：

Set-Cookie: name=Nicholas; expires=Sat, 02 May 2009 23:38:25 GMT



没有设置 expires 选项时，cookie 的生命周期仅限于当前会话中，关闭浏览器意味着这次会话的结束，所以会话 cookie 仅存在于浏览器打开状态之下。这就是为什么为什么当你登录一个 Web 应用时经常会看到一个复选框，询问你是否记住登录信息：如果你勾选了复选框，那么一个 expires 选项会被附加到登录 cookie 中。如果 expires 设置了一个过去的时间点，那么这个 cookie 会被立即删掉。





**domain 选项**

domain指定了 cookie 将要被发送至哪个或哪些域中。默认情况下，domain会被设置为创建该 cookie 的页面所在的域名，所以当给相同域名发送请求时该 cookie 会被发送至服务器。domain 选项可用来扩充 cookie 可发送域的数量，例如：

Set-Cookie: name=Nicholas; domain=nczonline.net



像 Yahoo! 这种大型网站，都会有许多 name.yahoo.com 形式的站点（例如：my.yahoo.com, finance.yahoo.com 等等）。将一个 cookie 的 domain 选项设置为 yahoo.com，就可以将该 cookie 的值发送至所有这些站点。浏览器会把 domain 的值与请求的域名做一个尾部比较（即从字符串的尾部开始比较），并将匹配的 cookie 发送至服务器。

domain 选项的值必须是发送 Set-Cookie 消息头的主机名的一部分，例如我不能在 google.com 上设置一个 cookie，因为这会产生安全问题。不合法的 domain 选择将直接被忽略。



**path 选项**



path选项指定了请求的资源 URL 中必须存在指定的路径时，才会发送Cookie 消息头。这个比较通常是将 path 选项的值与请求的 URL 从头开始逐字符比较完成的。如果字符匹配，则发送 Cookie 消息头，例如：

Set-Cookie:name=Nicholas;path=/blog

在这个例子中，path 选项值会与 /blog，/blogrool 等等相匹配；任何以 /blog 开头的选项都是合法的。

需要注意的是，只有在 domain 选项核实完毕之后才会对 path 属性进行比较。path 属性的默认值是发送 Set-Cookie 消息头所对应的 URL 中的 path 部分。







**secure 选项**

最后一个选项是 secure。不像其它选项，该选项只是一个标记而没有值。只有当一个请求通过 SSL 或 HTTPS 创建时，包含 secure 选项的 cookie 才能被发送至服务器。这种 cookie 的内容具有很高的价值，如果以纯文本形式传递很有可能被篡改，例如：

Set-Cookie: name=Nicholas; secure

事实上，机密且敏感的信息绝不应该在 cookie 中存储或传输，因为 cookie 的整个机制原本都是不安全的。默认情况下，在 HTTPS 链接上传输的 cookie 都会被自动添加上 secure 选项。



**Cookie 的维护和生命周期**



在一个 cookie 中可以指定任意数量的选项，并且这些选项可以是任意顺序，例如：

Set-Cookie:name=Nicholas; domain=nczonline.net; path=/blog

这个 cookie 有四个标识符：cookie 的 name，domain，path，secure 标记。要想改变这个 cookie 的值，需要发送另一个具有相同 cookie name，domain，path 的 Set-Cookie 消息头。例如：

Set-Cookie: name=Greg; domain=nczonline.net; path=/blog

这将覆盖原来 cookie 的值。





但是，修改 cookie 选项的任意一项都将创建一个完全不同的新 cookie，例如：

Set-Cookie: name=Nicholas; domain=nczonline.net; path=/

这个消息头返回之后，会同时存在两个名为 “name” 的不同的 cookie。如果你访问 www.nczonline.net/blog 下的一个页面，以下的消息头将被包含进来：

Cookie: name=Greg; name=Nicholas

在这个消息头中存在了两个名为 “name” 的 cookie，path 值越详细则 cookie 越靠前。 按照 domain-path-secure 的顺序，设置越详细的 cookie 在字符串中越靠前。假设我在 ww.nczonline.net/blog 下用默认选项创建了另一个 cookie：

Set-Cookie: name=Mike

那么返回的消息头现在则变为：

Cookie: name=Mike; name=Greg; name=Nicholas

以 “Mike” 作为值的 cookie 使用了域名（www.nczonline.net）作为其 domain 值并且以全路径（/blog）作为其 path 值，则它较其它两个 cookie 更加详细。





**cookie 自动删除**



cookie 会被浏览器自动删除，通常存在以下几种原因：

- 会话 cooke (Session cookie) 在会话结束时（浏览器关闭）会被删除
- 持久化 cookie（Persistent cookie）在到达失效日期时会被删除
- 如果浏览器中的 cookie 数量达到限制，那么 cookie 会被删除以为新建的 cookie 创建空间。详见我的另外一篇关于 [cookies restrictions](http://www.nczonline.net/blog/2008/05/17/browser--restrictions/) 的博客

对于自动删除来说，Cookie 管理显得十分重要，因为这些删除都是无意识的。





**HTTP-Only cookies**



微软的 IE6 SP1 在 cookie 中引入了一个新的选项：HTTP-only，HTTP-Only 背后的意思是告之浏览器该 cookie 绝不能通过 JavaScript 的 document.cookie 属性访问。

今天 Firefox2.0.0.5+、Opera9.5+、Chrome 都支持 HTTP-Only cookie。3.2 版本的 Safari 仍不支持。

要创建一个 HTTP-Only cookie，只要向你的 cookie 中添加一个 HTTP-Only 标记即可：

Set-Cookie: name=Nicholas; HttpOnly



你不能通过 JavaScript 设置 HTTP-only，因为你不能再通过 JavaScript 读取这些 cookie，这是情理之中的事情。







https://javascript.ruanyifeng.com/bom/cookie.html

https://studygolang.com/articles/5905