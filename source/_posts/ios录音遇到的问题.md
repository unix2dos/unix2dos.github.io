---
title: iOS录音遇到的问题
tags: iOS
abbrlink: 9ecea432
date: 2017-02-20 09:54:54
---
### iOS使用openAL控制声音的输出设备
项目中播放ios录音的时候使用的是AVAudio相关库, 播放音效又是用的openAL.
如果同时或交替播放这两类声音, 会造成声音一会从听筒发声,一会从扬声器发声.
千辛万苦找到解决方案:

```cpp
Interesting enough, it can be done!

Basically you add a property listener to get route change events:
    AudioSessionAddPropertyListener(kAudioSessionProperty_AudioRouteChange, audioRouteChangeListenerCallback,0);

	Then in the callback, determine if its a headphone being plugged-in and override the audio route:
	        UInt32 audioRouteOverride = kAudioSessionOverrideAudioRoute_Speaker;
			        AudioSessionSetProperty(kAudioSessionProperty_OverrideAudioRoute, sizeof(audioRouteOverride), &audioRouteOverride);

					Too simple...
```


### iOS AVAudioSession 监听静音开关
录音使用AVAudioSession播放的时候, 无法识别Iphone手机的物理静音开关,需要修改下模式


```
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
```

修改成
        
```
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategorySoloAmbient error:nil];//监听静音
```



