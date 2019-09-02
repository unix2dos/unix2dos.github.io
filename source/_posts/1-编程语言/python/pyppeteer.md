---
title: python爬虫利器pyppeteer的使用
tags:
  - python
  - pyppeteer
  - 爬虫
categories:
  - 1-编程语言
  - python
abbrlink: 6b1ba1b4
date: 2019-08-31 12:00:46
---



### 0. 前言

Chrome59(linux、macos)、 Chrome60(windows)之后，Chrome自带[headless(无界面)模式](https://developers.google.com/web/updates/2017/04/headless-chrome)很方便做自动化测试或者爬虫。但是如何和headless模式的Chrome交互则是一个问题。通过启动Chrome时的命令行参数仅能实现简易的启动时初始化操作。Selenium、Webdriver等是一种解决方案，但是往往依赖众多，不够扁平。



puppeteer是谷歌官方出品的一个通过DevTools协议控制headless Chrome的Node库。可以通过puppeteer的提供的api直接控制Chrome模拟大部分用户操作来进行UI Test或者作为爬虫访问页面来收集数据。



pyperteer是puppeteer的Python实现，相比于selenium具有异步加载、速度快、具备有界面/无界面模式、伪装性更强不易被识别为机器人同时可以伪装手机平板等终端；但是也有一些缺点，如接口不易理解、语义晦涩；

<!-- more -->



### 1. pyppeteer使用

```bash
pip3 install pyppeteer
```





##### 1.1 无头模式

```python
browser = await launch({'headless': False, 'args': ['--no-sandbox']})
```

headless=True, 不弹出浏览器, 测试阶段可以设置为 False 观测

##### 1.2 page方法

```bash
page.click 点击
page.type 输入
page.select 下拉框


page.click('.rfm input[name="cookietime"]')
```

对于 page方法内的选择元素语法请参考:  https://www.w3schools.com/cssref/css_selectors.asp



### 2. pyppeteer  linux运行问题

##### 2.1 centos无法运行pyppeteer

```
yum -y install libX11 libXcomposite libXcursor libXdamage libXext libXi libXtst cups-libs libXScrnSaver libXrandr alsa-lib pango atk at-spi2-atk gtk3 
```



##### 2.2 Bad NaCl helper startup ack

```
ERROR:nacl_fork_delegate_linux.cc(314)] Bad NaCl helper startup ack (0 bytes)\n\n(chrome:24935)
```

使用无头模式



##### 2.3 Navigation Timeout Exceeded: 30000 ms exceeded

```
await page.goto("https://www.baidu.com", timeout=0)
```

+ 加上timeout

+ 检测封禁 ip





### 3. pyppeteer 问题解决方案

##### 3.1 抓取js 渲染后的数据

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



##### 3.2 webdriver 检测问题

有些网站还是会检测到是 webdriver 吧，比如淘宝检测到是 webdriver 就会禁止登录了

其实淘宝主要通过 window.navigator.webdriver 来对 webdriver 进行检测，所以我们只需要使用 JavaScript 将它设置为 false 即可，代码如下：

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



##### 3.3 保持用户记录

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



### 4. 参考资料

+ https://github.com/miyakogi/pyppeteer
+ [Python爬虫入门教程 24-100 微医挂号网医生数据抓取](https://juejin.im/post/5c35944b6fb9a049de6d8dd2)
+ [Python中与selenium齐名的pyppeteer库](https://zhuanlan.zhihu.com/p/63634783)
+ [pyppeteer使用遇到的bug及解决方法](https://www.sanfenzui.com/pyppeteer-bug-collection.html)
+ https://juejin.im/post/59e5a86c51882578bf185dba