slack以及机器人slack-hubot

 

 

 

 




在国内, 为什么不选用钉钉, 飞书, 企业微信三大巨头?
slack 是谁? 一个标杆

并且还没有被墙

hobut 更是吊打上面三个




npm install -g yo generator-hubot




mkdir my-awesome-hubot && cd my-awesome-hubot
yo hubot --adapter=slack  #回答问题





1. 注意不要用官网最新的教程

   坑爹!!

2. 用这个

   https://github.com/slackapi/hubot-slack/issues/584#issuecomment-611808704











注意 token是以xoxb-开头的









```bash
HUBOT_SLACK_TOKEN=xoxb-YOUR-TOKEN-HERE ./bin/hubot --adapter slack
```



HUBOT_SLACK_TOKEN=xoxb-1799820628257-1794120811364-AjZM7HFtI6tGMmjGl0Wij8Ae ./bin/hubot --adapter slack







# 3. 参考资料

+ https://slack.dev/hubot-slack/
+ https://github.com/slackapi/hubot-slack/issues/584

