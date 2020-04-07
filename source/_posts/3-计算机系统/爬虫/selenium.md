---
title: 爬虫利器selenium和无头浏览器的使用
tags:
  - 爬虫
  - python
categories:
  - 3-计算机系统
  - 爬虫
abbrlink: 545fa06
date: 2019-09-04 22:04:46
---



### 0. 前言

Selenium 的初衷是打造一款优秀的自动化测试工具，但是慢慢的人们就发现，Selenium 的自动化用来做爬虫正合适。我们知道，传统的爬虫通过直接模拟 HTTP 请求来爬取站点信息，由于这种方式和浏览器访问差异比较明显，很多站点都采取了一些反爬的手段，而 Selenium 是通过模拟浏览器来爬取信息，其行为和用户几乎一样，反爬策略也很难区分出请求到底是来自 Selenium 还是真实用户。



通过 Selenium 来做爬虫，不用去分析每个请求的具体参数，比起传统的爬虫开发起来更容易。Selenium 爬虫唯一的不足是慢，如果你对爬虫的速度没有要求，那使用 Selenium 是个非常不错的选择。

<!-- more -->

### 1. 安装和使用

```bash
pip install selenium 
```



##### 1.1 页面操作

```python
# 输入数据
browser.find_element_by_css_selector('.rfm input[name="username"]').send_keys('123456')

# 选择数据
browser.find_element_by_xpath("//select[@name='questionid']/option[text()='父亲的手机号码']").click()

# 敲回车
browser.find_element_by_css_selector('button[name="loginsubmit"]').send_keys(Keys.ENTER)


# 只等3秒
browser.implicitly_wait(3)

# 获取cookie
cookies_list = driver.get_cookies()
cookies_dict = {}
for cookie in cookies_list:
    cookies_dict[cookie['name']] = cookie['value']
print(cookies_dict)
```



### 2. 遇到的问题



##### 2.1 [ImportError: cannot import name 'webdriver'](https://stackoverflow.com/questions/29092970/importerror-cannot-import-name-webdriver)

文件不能命名为`selenium`



##### 2.2  Message: 'chromedriver' executable needs to be in PATH.

下载驱动 https://sites.google.com/a/chromium.org/chromedriver/downloads



##### 2.3 Message: unknown error: cannot find Chrome binary

+ centos 安装 chrome

  ```bash
  wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
  sudo yum localinstall google-chrome-stable_current_x86_64.rpm
  google-chrome --no-sandbox --version # 看到版本后去下载相关的driver
  ```

+ ubuntu 安装 chrome

  ```
  wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
  sudo apt install ./google-chrome-stable_current_amd64.deb
google-chrome --no-sandbox --version
  ```
  
  



### 3. 参考资料

+ https://cuiqingcai.com/2599.html

+ [Python3中Selenium使用方法](https://zhuanlan.zhihu.com/p/29435831)


+ [使用 Python + Selenium 打造浏览器爬虫](https://www.aneasystone.com/archives/2018/02/python-selenium-spider.html)