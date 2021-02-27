---
title: slack介绍和机器人使用
tags:
  - slack
categories:
  - 5-随笔
  - 个人记录
date: 2021-02-27 00:00:00
---

经常用的 bearychat 凉了(估计受疫情影响),  还有国内的瀑布IM也凉了, 不得不选用一个新的企业协作工具.

那么在国内为什么不选用钉钉, 飞书, 企业微信? 问就是我不配. slack 是谁? 前面三个的标杆

<!-- more -->

# 1. 使用

### 1.1  机器人安装

注意先不要用官网最新的教程, 有坑!!!!  看这个 https://github.com/slackapi/hubot-slack/issues/584#issuecomment-611808704

+ Create a classic app from https://api.slack.com/apps?new_classic_app=1
+ Go to **Features** > **OAuth & Permissions** > **Scopes**
  + Click "Add an OAuth Scope"
  + Search "bot" and choose it
+ Go to **Features** > **App Home**
  + Click "Add Legacy Bot User"
  + Input "Display Name" and "Default username"
  + Click "Add"
+ Go to **Settings** > **Install App**
  + Click "Install App to Workspace"
  + Complete the OAuth flow



### 1.2 字段说明

+ token

  https://api.slack.com/apps?new_classic_app=1  

  1. 点击 app 进去

  2. Features > OAuth & Permissions

     看到 User OAuth Token && Bot User OAuth Token

+ teamid 和 channelId

  以一个地址为例:  https://app.slack.com/client/T01PHQ4J7K/C01PL8L0Y65

  T 开头的是 teamID

  C 开头的是 channleID



### 1.3 机器人发送信息

+ 不建议使用 web hookapi, 因为 channel 过多的时候会很累, 并且功能也少

+ 注意 token 是机器人的 token, 即`Bot User OAuth Token`

+ 注意需要把bot 拉入channel 里, `/invite @BOT_NAME`

+ golang 代码

  https://github.com/slack-go/slack/blob/master/examples/messages/messages.go

``` go
package main

import (
	"fmt"

	"github.com/slack-go/slack"
)

func main() {
	api := slack.New("YOUR_TOKEN_HERE")

	attachment := slack.Attachment{
		Pretext: "some pretext",
		Text:    "some text",
		// Uncomment the following part to send a field too
		/*
			Fields: []slack.AttachmentField{
				slack.AttachmentField{
					Title: "a",
					Value: "no",
				},
			},
		*/
	}

	channelID, timestamp, err := api.PostMessage(
		"CHANNEL_ID",
		slack.MsgOptionText("Some text", false),
		slack.MsgOptionAttachments(attachment),
		slack.MsgOptionAsUser(true), // Add this if you want that the bot would post message as a user, otherwise it will send response using the default slackbot
	)
	if err != nil {
		fmt.Printf("%s\n", err)
		return
	}
	fmt.Printf("Message successfully sent to channel %s at %s", channelID, timestamp)
}
```



# 2. 参考资料

+ https://github.com/slackapi/hubot-slack/issues/584 创建机器人
+ https://app.slack.com/block-kit-builder/ 设计 msg
+ https://github.com/slack-go/slack

