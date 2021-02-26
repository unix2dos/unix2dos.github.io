在国内, 为什么不选用钉钉, 飞书, 企业微信三大巨头?
slack 是谁? 一个标杆

并且还没有被墙, bot 更是吊打上面三个



# 1. 安装

```
npm install -g yo generator-hubot

mkdir my-awesome-hubot && cd my-awesome-hubot
yo hubot --adapter=slack  #回答问题
```

1. 注意不要用官网最新的教程

   坑爹!!

2. 用这个

   https://github.com/slackapi/hubot-slack/issues/584#issuecomment-611808704

3. 注意 token是以xoxb-开头的

  ```bash
  HUBOT_SLACK_TOKEN=xoxb-YOUR-TOKEN-HERE ./bin/hubot --adapter slack
  ```



# 2. bot

### 2.1 income

https://api.slack.com/messaging/webhooks



curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' https://hooks.slack.com/services/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





### 2.2 制作

https://app.slack.com/block-kit-builder/















# 3. 参考资料

+ https://slack.dev/hubot-slack/
+ https://github.com/slackapi/hubot-slack/issues/584

