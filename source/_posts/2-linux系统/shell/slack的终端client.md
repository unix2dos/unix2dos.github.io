---
title: slack的终端client
tags:
  - slack
categories:
  - 2-linux系统
  - shell
date: 2021-03-10 00:00:00
---

# 1. slack term

让 slack 运行在终端里, 可以更快的看到消息并回复.

![Screenshot](slack%E7%9A%84%E7%BB%88%E7%AB%AFclient/screenshot.png)

<!-- more -->

# 2. 安装和使用

### 2.1 安装

参考: https://github.com/erroneousboat/slack-term#installation

+ 下载最新的 release 二进制

+ 执行

  ```bash
  mv slack-term-darwin-amd64 /usr/local/bin/slack-term

  slack-term --config ~/.config/slack-term/config
  ```

+ 注意, 默认执行`slack-term`  并没有生成配置, 所以需要带`-- config` 参数



### 2.2 配置

```bash
alias slack="slack-term --config ~/.config/slack-term/config"
```

获取 token: https://github.com/erroneousboat/slack-term/wiki#running-slack-term-without-legacy-tokens



`vi ~/.config/slack-term/config`

```ini
{
	"slack_token": "xoxp-",
	"emoji": true
}
```

# 3. 参考资料:

+ https://github.com/erroneousboat/slack-term