---
title: xcode自定义Eclipse中常用的快捷键
tags:
  - xcode
  - iOS
abbrlink: 30321ed4
categories:
  - mac工具使用
date: 2016-12-14 17:54:46
---



首先找到Xcode中的自带的配置文件

```
/Applications/Xcode.app/Contents/Frameworks/IDEKit.framework/Versions/A/Resources/IDETextKeyBindingSet.plist
```
这个文件里配置了一些可以设置快捷键的操作, 使用常用的编辑器打开它（需要root权限）。

```
	<key>GDI Commands</key>
	<dict>
		<key>GDI Duplicate Current Line</key>
		<string>selectLine:, copy:, moveToEndOfLine:, insertNewline:, paste:, deleteBackward:</string>
		<key>GDI Delete Current Line</key>
		<string>deleteToBeginningOfLine:, moveToEndOfLine:, deleteToBeginningOfLine:, deleteBackward:, moveDown:, moveToBeginningOfLine:</string>
		<key>GDI Move Current Line Up</key>
		<string>selectLine:, cut:, moveUp:, moveToBeginningOfLine:, insertNewLine:, paste:, moveBackward:</string>
		<key>GDI Move Current Line Down</key>
		<string>selectLine:, cut:, moveDown:, moveToBeginningOfLine:, insertNewLine:, paste:, moveBackward:</string>
		<key>GDI Insert Line Above</key>
		<string>moveUp:, moveToEndOfLine:, insertNewline:</string>
		<key>GDI Insert Line Below</key>
		<string>moveToEndOfLine:, insertNewline:</string>
	</dict>
```
<!-- more -->
把这段配置放到上面提到的IDETextKeyBindingSet.plist里，放在文件的最后的这两行之前：
</dict>
</plist>

重启Xcode，在Xcode菜单中，打开Preferences，选中Key Binding，在右上方搜索GDI, 会出现类似下图的显示，如果没有的话，请检查上面的每步操作。

