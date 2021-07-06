---
title: slack的终端client
tags:
  - slack
categories:
  - 2-linux系统
  - shell
abbrlink: b47a642f
date: 2021-03-10 00:00:00
---

# 1. slack terminal client

让 slack 运行在终端里, 可以更快的看到消息并回复

![Screenshot](slack%E7%9A%84%E7%BB%88%E7%AB%AFclient/screenshot.png)

<!-- more -->

# 2. 安装和使用

### 2.1 安装

下载最新的release:  https://github.com/erroneousboat/slack-term/releases

```bash
mv slack-term-darwin-amd64 /usr/local/bin/slack-term

slack-term --config ~/.config/slack-term/config # slack-term并没有生成默认配置, 所以需要带--config指定文件
```



### 2.2 配置

`vi ~/.config/slack-term/config`

```ini
{
	"slack_token": "xoxp-",
	"emoji": true
}
```

获取token地址:  https://github.com/erroneousboat/slack-term/wiki#running-slack-term-without-legacy-tokens

最后创建一个别名:

```bash
alias slack="slack-term --config ~/.config/slack-term/config"
```



# 3. 参考资料

+ https://github.com/erroneousboat/slack-term