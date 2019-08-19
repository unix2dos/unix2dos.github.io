Chrome59(linux、macos)、 Chrome60(windows)之后，Chrome自带[headless(无界面)模式](https://developers.google.com/web/updates/2017/04/headless-chrome)很方便做自动化测试或者爬虫。但是如何和headless模式的Chrome交互则是一个问题。通过启动Chrome时的命令行参数仅能实现简易的启动时初始化操作。Selenium、Webdriver等是一种解决方案，但是往往依赖众多，不够扁平。



Puppeteer是谷歌官方出品的一个通过DevTools协议控制headless Chrome的Node库。可以通过Puppeteer的提供的api直接控制Chrome模拟大部分用户操作来进行UI Test或者作为爬虫访问页面来收集数据。



pyperteer是puppeteer的Python实现，相比于selenium具有异步加载、速度快、具备有界面/无界面模式、伪装性更强不易被识别为机器人同时可以伪装手机平板等终端；但是也有一些缺点，如接口不易理解、语义晦涩；





### pyppeteer

```python
browser = await launch({'headless': False, 'args': ['--no-sandbox']})
```

headless=True, 不弹出浏览器



page.click 点击
page.type 输入
page.select 下拉框



### pyppeteer page selectors

```python
page.click('.rfm input[name="cookietime"]')
```

+ https://www.w3schools.com/cssref/css_selectors.asp



### pgquery

```python
    def parse_html(self,content):
        doc = pq(content)
        items = doc(".dt").items()
        for item in items:
            if item.find("center").text() == "":
                continue

            title = item.find("center").text()

            for i in item.find("th").items():
                category = i.find("a").eq(0).text()
                neirong = i.find("a").eq(1).text()
                url = i.find("a").eq(1).attr('href')

                one_data = {
                    "category": category,
                    "context": neirong,
                    "url": url,
                }
                print(one_data)
                if title == "最新活动信息" :
                    self.activityData.append(one_data)
                else:
                    self.infoData.append(one_data)
```



### 打开文件并设置编码

```
    with open("1.html", "r", encoding='gbk') as f:
        contents = f.read()
        zk8.parse_html(contents)
```





### centos无法运行 pyppeteer的解决方案

```
yum -y install libX11 libXcomposite libXcursor libXdamage libXext libXi libXtst cups-libs libXScrnSaver libXrandr alsa-lib pango atk at-spi2-atk gtk3 
```





### 3. 参考资料

+ https://github.com/miyakogi/pyppeteer
+ https://juejin.im/post/5c35944b6fb9a049de6d8dd2
+ https://zhuanlan.zhihu.com/p/63634783
+ https://www.sanfenzui.com/pyppeteer-bug-collection.html