---
title: 微信机器人itchat的使用
tags:
  - wechat
  - python
  - robot
abbrlink: 4f93001b
categories:
  - 计算机系统
  - 爬虫
  - 机器人
date: 2019-03-31 12:01:00
---



近期准备用微信机器人实现往微信群里发消息. 需要用到微信机器人.

目前的微信机器人大部分都是基于web微信协议, 因此仅能覆盖 Web 微信本身所具备的功能。例如收发消息, 加好友, 转发消息, 自动回复, 陪人聊天,消息防撤回等等.

但是web微信目前不支持抢红包和朋友圈等相关功能, 并且使用机器人存在一定概率被限制登录的可能性, 主要表现为无法登陆 Web 微信 (但不影响手机等其他平台)。

<!-- more -->



### 1. 安装使用

+ 可参考: https://github.com/littlecodersh/ItChat

```shell
pip3 install itchat
```



+ 登录微信并且向文件助手发送一条消息

```python
import itchat

itchat.auto_login()

itchat.send('Hello, filehelper', toUserName='filehelper')
```



+ 更多例子可以参考官方文档 https://itchat.readthedocs.io/zh/latest/



### 2. 发送消息到微信群

发送消息到微信群, 首先要保证微信群保存在通讯录, 如果不保存到通讯录，是无法在各设备之间同步的（所以itchat也无法读取到）

```python
# coding=utf8
import itchat


def send_group(group, msg):
    rooms = itchat.get_chatrooms(update=True)
    rooms = itchat.search_chatrooms(name=group)
    if not rooms:
        print("None group found")
    else:
        itchat.send(msg, toUserName=rooms[0]["UserName"])


if __name__ == "__main__":
    itchat.auto_login(hotReload=True)
    send_group(u"你的群聊名字", "test msg")
    itchat.run()

```





### 3. 机器人AI聊天

```python
# coding=utf8
import itchat
import requests

KEY = 'xxxxxxxx' #可以去http://www.turingapi.com/申请


def get_response(msg):
    apiUrl = 'http://www.tuling123.com/openapi/api'
    data = {
        'key': KEY,
        'info': msg,
        'userid': 'wechat-robot',
    }
    try:
        r = requests.post(apiUrl, data=data).json()
        return r.get('text')
    except:
        return


@itchat.msg_register(itchat.content.TEXT)
def tuling_reply(msg):
    defaultReply = 'I received: ' + msg['Text']
    reply = get_response(msg['Text'])
    return reply or defaultReply


itchat.auto_login(hotReload=True)
itchat.run()
```



### 4. 消息防撤回

可参考下面的这个项目

https://github.com/ccding/wechat-anti-revoke 



### 5. 其他微信机器人项目

+ https://github.com/youfou/wxpy   在 itchat 的基础上，通过大量接口优化提升了模块的易用性，并进行丰富的功能扩展
+ https://github.com/lb2281075105/Python-WeChat-ItChat 使用itchat的一些demo
+ https://github.com/littlecodersh/itchatmp 微信公众号、企业号接口项目
+ https://github.com/newflydd/itchat4go golang版本封装的itchat



