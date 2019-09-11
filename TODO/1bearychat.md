





```python
 def channel_id(self, name):
        lists = self.client.channel.list()
        for x in lists:
            if x["name"] == name:
                return x["vchannel_id"]


    def msg_create(self, msg):
        id = self.channel_id("a")
        data = {
            "vchannel_id": id,
            "markdown": True,
            "text": msg.text,
            "attachments": [
                {
                    "title": msg.text,
                    "url": msg.url,
                    "text": msg.text,
                    "color": "#ffffff",
                    "images": [
                        {"url": "http://img3.douban.com/icon/ul15067564-30.jpg"}
                    ]
                }
            ]
        }
        self.client.message.create(json=data)
```

