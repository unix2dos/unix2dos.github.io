



### 1. 网络请求



requests urllib的对比



import requests

url = 'http://www.baidu.com'

response = requests.get(url)

html = response.text

print(html)





import requests

url = "http://docs.python-requests.org/zh_CN/latest/_static/requests-sidebar.png"

response = requests.get(url)

with open('image.png','wb') as f:

  f.write(response.content)







### 2. 数据提取 (考虑pyquery)



规则

json





不规则

字符串匹配 和 正则表达式



html xml 

beautifulsoup



pip install beautifulsoup4



支持不同的解释器



pip install lxml

pip install html5lib





BeautifulSoup(markup, “html.parser”)	# 原生

BeautifulSoup(markup, “lxml”)	 # 一款解析 html

BeautifulSoup(markup, “xml”)  # 解析 xml

BeautifulSoup(markup, “html5lib”)  # 另外一款解析 html









Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象





lxml 



pip install lxml



xpath语法



html



from lxml import etree

text = '''

<div>

​    <ul>

​         <li class="item-0"><a href="link1.html">first item</a></li>

​         <li class="item-1"><a href="link2.html">second item</a></li>

​         <li class="item-inactive"><a href="link3.html">third item</a></li>

​         <li class="item-1"><a href="link4.html">fourth item</a></li>

​         <li class="item-0"><a href="link5.html">fifth item</a>

​     </ul>

 </div>

'''

html = etree.HTML(text)

result = etree.tostring(html)

print(result)





自动修正 html 语法. 不仅补全了 li 标签，还添加了 body，html 标签。







xpath, beautfoul 对比



BeautifulSoup是一个库，而XPath是一种技术，python中最常用的XPath库是lxml，因此，这里就拿lxml来和BeautifulSoup做比较吧





**1 性能 lxml >> BeautifulSoup**

BeautifulSoup和lxml的原理不一样，BeautifulSoup是基于DOM的，会载入整个文档，解析整个DOM树，因此时间和内存开销都会大很多。而lxml只会局部遍历，另外lxml是用c写的，而BeautifulSoup是用python写的，因此性能方面自然会差很多。



**2 易用性 BeautifulSoup >> lxml**

BeautifulSoup用起来比较简单，API非常人性化，支持css选择器。lxml的XPath写起来麻烦，开发效率不如BeautifulSoup。

title = soup.select('.content div.title h3')

同样的代码用Xpath写起来会很麻烦

title = tree.xpath("//*[@class='content']/div[@class='content']/h3")





### 3. PhantomJS(暂停开发)



brew cask install phantomjs





https://phantomjs.org/download.html  PhantomJS宣布暂停开发。





serWarning: Selenium support for PhantomJS has been deprecated, please use headless versions of Chrome or Firefox instead



新版本的Selenium不再支持PhantomJS了，请使用Chrome或Firefox的无头版本来替代。









PhantomJS是一个无界面的,可脚本编程的WebKit浏览器引擎。它原生支持多种web 标准：DOM 操作，CSS选择器，JSON，Canvas 以及SVG。因此可以比浏览器更加快速的解析处理js加载。

​	

有时，我们需要浏览器处理网页，但并不需要浏览，比如生成网页的截图、抓取网页数据等操作。[PhantomJS](http://phantomjs.org/)的功能，就是提供一个浏览器环境的命令行接口，你可以把它看作一个“虚拟浏览器”，除了不能浏览，其他与正常浏览器一样。它的内核是WebKit引擎，不提供图形界面，只能在命令行下使用，我们可以用它完成一些特殊的用途。



然后把二进制放到一个目录下, 增加个$PATH 指定即可

phantomjs -v



以前写爬虫，遇到需要登录的页面，一般都是通过chrome的检查元素，查看登录需要的参数和加密方法，如果网站的加密非常复杂，例如登录qq的，就会很蛋疼。



现在有了PhantomJS，再也不需要考虑登录的参数和加密了，用PhantomJS打开页面，通过JS或JQuery语句，填入账号和密码，然后点击登陆，然后把Cookies保存下来，就可以模拟登陆了。



### 4. Selenium(考虑Pyppeteer)

Selenium 是什么？一句话，自动化测试工具。它支持各种浏览器，包括 Chrome，Safari，Firefox 等主流界面式浏览器，如果你在这些浏览器里面安装一个 Selenium 的插件，那么便可以方便地实现Web界面的测试。换句话说叫 Selenium 支持这些浏览器驱动。话说回来，PhantomJS不也是一个浏览器吗，那么 Selenium 支持不？答案是肯定的，这样二者便可以实现无缝对接了。



嗯，所以呢？安装一下 Python 的 Selenium 库，再安装好 PhantomJS，不就可以实现 Python＋Selenium＋PhantomJS 的无缝对接了嘛！PhantomJS 用来渲染解析JS，Selenium 用来驱动以及与 Python 的对接，Python 进行后期的处理，完美的三剑客！



有人问，为什么不直接用浏览器而用一个没界面的 PhantomJS 呢？答案是：效率高！



用Selenium驱动无头浏览器



走另外一个地址





### 5. PyQuery 



pyquery 可让你用 jQuery 的语法来对 xml 进行操作。这I和 jQuery 十分类似。如果利用 lxml，pyquery 对 xml 和 html 的处理将更快。

这个库不是（至少还不是）一个可以和 JavaScript交互的代码库，它只是非常像 jQuery API 而已。



pip install pyquery





### 6. Pyppeteer 

Pyppeteer 就是依赖于 Chromium 这个浏览器来运行的。那么有了 Pyppeteer 之后，我们就可以免去那些繁琐的环境配置等问题。如果第一次运行的时候，Chromium 浏览器没有安全，那么程序会帮我们自动安装和配置，就免去了繁琐的环境配置等工作。另外 Pyppeteer 是基于 Python 的新特性 async 实现的，所以它的一些执行也支持异步操作，效率相对于 Selenium 来说也提高了。



pip3 install pyppeteer





使用 requests 是无法正常抓取到相关数据的。因为什么？因为这个页面是 JavaScript 渲染而成的，我们所看到的内容都是网页加载后又执行了 JavaScript 之后才呈现出来的，因此这些条目数据并不存在于原始 HTML 代码中，而 requests 仅仅抓取的是原始 HTML 代码。



好的，所以遇到这种类型的网站我们应该怎么办呢？

其实答案有很多：

- 分析网页源代码数据，如果数据是隐藏在 HTML 中的其他地方，以 JavaScript 变量的形式存在，直接提取就好了。
- 分析 Ajax，很多数据可能是经过 Ajax 请求时候获取的，所以可以分析其接口。
- 模拟 JavaScript 渲染过程，直接抓取渲染后的结果。



而 Pyppeteer 和 Selenium 就是用的第三种方法，下面我们再用 Pyppeteer 来试试，如果用 Pyppeteer 实现如上页面的抓取的话，代码就可以写为如下形式：

```python
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq

async def main():
    browser = await launch()
    page = await browser.newPage()
    await page.goto('http://quotes.toscrape.com/js/')
    doc = pq(await page.content())
    print('Quotes:', doc('.quote').length)
    await browser.close()

asyncio.get_event_loop().run_until_complete(main())
```







```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch(devtools=True)
    page = await browser.newPage()
    await page.goto('https://www.baidu.com')
    await asyncio.sleep(100)

asyncio.get_event_loop().run_until_complete(main())
```





有些网站还是会检测到是 webdriver 吧，比如淘宝检测到是 webdriver 就会禁止登录了，我们可以试试：





OK，那刚才所说的 webdriver 检测问题怎样来解决呢？其实淘宝主要通过 window.navigator.webdriver 来对 webdriver 进行检测，所以我们只需要使用 JavaScript 将它设置为 false 即可，代码如下：

```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch(headless=False, args=['--disable-infobars'])
    page = await browser.newPage()
    await page.goto('https://login.taobao.com/member/login.jhtml?redirectURL=https://www.taobao.com/')
    await page.evaluate(
        '''() =>{ Object.defineProperties(navigator,{ webdriver:{ get: () => false } }) }''')
    await asyncio.sleep(100)

asyncio.get_event_loop().run_until_complete(main())
```



很多朋友在每次启动 Selenium 或 Pyppeteer 的时候总是是一个全新的浏览器，那就是没有设置用户目录，如果设置了它，每次打开就不再是一个全新的浏览器了，它可以恢复之前的历史记录，也可以恢复很多网站的登录信息。那么这个怎么来做呢？很简单，在启动的时候设置 userDataDir 就好了，示例如下：

```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch(headless=False, userDataDir='./userdata', args=['--disable-infobars'])
    page = await browser.newPage()
    await page.goto('https://www.taobao.com')
    await asyncio.sleep(100)

asyncio.get_event_loop().run_until_complete(main())
```









### pyspider

pyspider上手更简单，操作更加简便，因为它增加了 WEB 界面，写爬虫迅速，集成了phantomjs，可以用来抓取js渲染的页面。

pip install pyspider



pyspider all



  File "/Users/liuwei/anaconda3/envs/py3.7/lib/python3.7/site-packages/pyspider/run.py", line 231
    async=True, get_object=False, no_input=False):
        ^
SyntaxError: invalid syntax



async在python3.7中是关键字，所以运行错误, 作者竟然合并了竟然没有打新版本, 直接放弃
https://github.com/binux/pyspider/issues/817





### Scrapy



pip install Scrapy



Scrapy自定义程度高，比 PySpider更底层一些，适合学习研究，需要学习的相关知识多，不过自己拿来研究分布式和多线程等等是非常合适的。





### 多线程

 thread 库

“Python下多线程是鸡肋，推荐使用多进程！”





### 多进程



multiprocessing





### 参考资料

+ https://cuiqingcai.com/1052.html
+ https://cuiqingcai.com/6942.html
+ https://github.com/Kr1s77/Python-crawler-tutorial-starts-from-zero