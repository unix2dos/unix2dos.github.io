



# 1. General

- **Startup：**设置系统启动时是否自启动。
- **Alfred Hotkey：**设置呼出 Alfred 的热键。我设置成了 Double Cmd。
- **Where are you：**这个设置比较特别，因为 Alfred 内置了常用网站搜索功能，在这里设置了你所在的国家后，Alfred 在搜索时会打开搜索网站对应国家的网站。

# 2. Features

### 2.1 Default Results

- **Essentials：**可以设置搜索「应用程序」、「联系人」

- **Extras:** 其它的还能查询「文件夹」、「文本文件」、「压缩文件」、「图片」、「AppleScript」等其它文件。 我全选上了.

- **Search all file types：**搜索所有文件类型，不过 Alfred 建议我们可以通过 `Open + 关键字` 或者 `Space（空格键）` 来查询文件或者文件夹，因为如果全部选中的话不但影响查询速度，还容易混淆查询结果。

- **Search Scope：**设置 Alfred 查询时会搜索的文件夹，我们在这里可以自己添加和删除文件夹。

- **Fallbacks：** 是设置如果没有查到结果使用 Google 还是其它网站来搜索结果。默认反馈结果为 Google、Amazon、Wikipedia 网页搜索。

### 2.2 File Search

##### 2.2.1 Search

- **Quick Search：**快速搜索，勾选该选项后，我们可以使用`‘（单引号）`或者`Space（空格键）`快速启用打开文件或者文件夹，功能类似于使用`Open + 关键字`。
- **Opening Files：**输入`open`打开文件或者文件夹。
- **Revealing Files：**输入`find`查询文件或者文件夹的位置。
- **Inside Files：**输入`in`查找文本文件内含有查询文字的文件（这个功能很强大啊）。

- **File Tags：**输入`tags`查询含有查询 tags（标签） 的文件或者文件夹。

- **Don‘t Show：**选择查询结果中不出现「邮件」、「书签」、「音乐」、「联系人」、「历史记录」等其它文件内容（注：如果需要更为复杂的结果过滤，则需要使用自定义结果过滤的 WorkFlow ）。
- **Result Limit：**自定义显示结果个数——更多的结果意味着更大的灵活性（flexibility），而更少的结果以为这更高的性能（performance）。



##### 2.2.2 Navigation

默认情况下，在 Alfred 中，`→` 为「显示动作面板」，`Command + ↓`为前往下一层文件夹，`Command + ↑`为前往上一层文件夹。

+  **Fuzzy Matching:** 模糊匹配。
+ **Shortcuts：** 我们可以设置使用 `←`和 `→` 为文件夹导航的快捷键，设置`return（回车键）`为在 Finder 中打开选中文件夹的快捷键。
+ **Previous Path：**先前路径，在此可以设置热键（默认为`Option + Command + /`）或关键词，来快捷地访问最近一次在 Alfred 中使用文件导航访问的路径。



##### 2.2.3 Buffer

通过`Option + ↑` 来将选中的文件夹或者文件加入到缓存，我们可以看到如果存在缓存的话 Alfred 搜索界面上会出现选中文件的小图标。

通过`Option + →`来批量处理缓存中的文件夹和文件。我们可以打开、发邮件、拷贝、移动、删除（嗯对了你是不是感觉到这个功能就是代替鼠标选中文件然后右键的功能）。

使用`Option + ↓`可以添加一个文件到缓存并移动到下一选择项。

使用`Option + ←`可以移除已添加的缓存项中的最后一项。



##### 2.2.4 Advanced

- **Copy Path：**复制路径，选中该选项后,如果使用了将目录拷贝至粘贴板的功能后会在目录前后加上单引号。
- **AppleScript：**AppleScript 脚本，选中该选项后可以使用`Command+O`来打开 AppleScript 编辑器，而 Alfred 默认的操作是直接执行脚本。
- **Performance：**在搜索外部存储文件时使用文件类型图标？（这个没有试过不知道是不是这个功能）。
- **Sorting：**这个设置应该是每次打开查询结果的文件后，更新文件的 「`kMDItemFSContentChangeDate`」 的值（具体作用不明，待 Google 之）。
- **Home Folder：**设置表示 home 文件夹字符，默认为 `~`。 



# 3. Web Search

这里当然是网站搜索的一些设置，我们可以使用 Alfred 默认的一些搜索功能，或者自己设置一些自定义搜索。图中可以看到已经设置了「亚马逊中国」、「亚马逊日本」、「Google」、「百度」、「BiliBili」、「Youku」等其它自定义查询。点击 **Add Custom Search** 后我们就可以自定义查询了。



在设置自定义查询界面中，主要设置有：

- **Search URL：**网站查询的 URL，每个网站的查询 URL 可以先通过网站查询功能，然后查看浏览器的地址栏就能知道了。当然查询内容使用 `{query}` 变量来代替。
- **Title：**标题，这个是设置在查询时 Alfred 查询主界面显示的提示文字。
- **Keyword：**查询关键字，尽量使用简短容易辨识的文字。
- **Validation：**有效性，这个是用来测试设置是否有效的。



# 4. Web Bookmarks





# 5. 功能集合



### Calculator（计算器）

计算器这个就不多说了，主要有两个功能，一个就是直接输入简单的加减运算，一个就是输入 `=` 来输入复杂的计算，支持许多高级的数学函数。

### Dictionary（字典）

字典功能其实使用的是 Mac 系统自带的字典，可以设置使用的字典和查询关键字，输入 di+关键字来查询中英字典





# 6. Clipboard History

基于隐私的考虑，Alfred 是默认关闭「剪切板历史」功能的，我个人设置的查看「剪切板历史」的热键是`双击 Control`，方便调出；对于普通用户来说，Alfred 的剪贴板功能已经完全够用了，无需重复购买 Paste 等剪贴板管理工具。



需要点开 KeepPlainText



### 6.1 history

- **Clipboard Histroy：**剪切板历史，用于设置粘贴板历史保存的时间（默认为 24 小时）。
- **Viewer Hotkey：**查看热键，用于设置打开粘贴板查看器的热键。
- **Viewer Keyword：**查看关键词，用于设置打开粘贴板查看器的关键字。
- **Snippet Keyword：**片段关键词，用于设置片段查询的关键字。
- **Clear Keyword：**清空关键词，用于设置清空粘贴板历史的关键字。
- **Ignore Apps：**忽略应用程序，用于设置忽略记录至粘贴板历史的应用程序。





### 6.2 Merging

这是一个神奇的功能：当我们复制了一段文本后，再选中另外一段文本后，通过使用 `Command ＋ 双击 C 键` 可以将当前选中的文本追加到第一次复制的文本后面。并且可以设置是使用空格、回车来分割不同的片段。



### 6.3 . Advanced

这里主要设置自动粘贴当前选中的记录和设置复制文本内容的最大字节。



# 7. Snippets



此功能主要是用于设置文本片段，便于快速输入。例如，实现快速输入地址、常用问候语、常用代码片段等：

- **Name：**文本片段名称
- **Keyword：**文本片段关键字
- **Snippet：**文本片段内容

使用时可以通过打开粘贴板浏览器根据名称和关键字查询，或者直接使用前面设置的片段查询关键字来查询。

- 使用 snip 关键字查询文本片段
- 查询到结果后直接回车便能将片段内容输入到当前激活的应用程序内





# 8. 脑力

Alfred一个很重要的命令操作就是：`↑`，可以调用上次的历史命令！



### 8.1 chrome 书签

### 8.6 粘贴板



### 8.2 增加百度查询

+ web Search 上添加百度链接，里面图标部分 http://baidu.com/favicon.ico直接去百度上下载，然后直接拖进来就行了

```bash
http://www.baidu.com/s?wd={query}

Search Baidu for 'query'
```

+ 点击Default Results下的Setup fallback results 下点击加号便能找到（另外我这个是破解版的，破解版和付费才能设置Setup fallback results）

### 8.3 In 定位到行数



### 8.4 结合 iterm2

https://github.com/alanhg/others-note/issues/25

前缀设置为$

更改命令从 `iTerm` 打开的方式很简单，在 `Application` 选择为 `Custom`（自定义），然后在下方的文本框输入下面的苹果脚本代码就可以：

```bash
on alfred_script(q)  
    if application "iTerm2" is running or application "iTerm" is running then  
        run script "  
            on run {q}  
                tell application \":Applications:iTerm.app\"  
                    activate  
                    try  
                        select first window  
                        set onlywindow to false  
                    on error  
                        create window with default profile  
                        select first window  
                        set onlywindow to true  
                    end try  
                    tell current session of the first window  
                        if onlywindow is false then  
                            tell split vertically with default profile  
                                write text q  
                            end tell  
                        end if  
                    end tell  
                end tell  
            end run  
        " with parameters {q}  
    else  
        run script "  
            on run {q}  
                tell application \":Applications:iTerm.app\"  
                    activate  
                    try  
                        select first window  
                    on error  
                        create window with default profile  
                        select first window  
                    end try  
                    tell the first window  
                        tell current session to write text q  
                    end tell  
                end tell  
            end run  
        " with parameters {q}  
    end if  
end alfred_script
```



### 8.5 词典设置

### 

### 8.6 配置同步



在 `Advanced` 选项卡中点击 `Set sync folder` 按钮把同步文件设置到iCloud中即可保持同步备份。

### 

# 9. 自定义的修改

+  设定->键盘->快捷键->聚焦->去掉快捷键，Spotlight快捷



# 11 . 扩展

### 11.1 http://www.packal.org/

# 

https://github.com/TKkk-iOSer/wechat-alfred-workflow



# 10. 参考资料

+ https://sspai.com/post/32979

+ https://juejin.cn/post/6844904062484217863

+ https://wellsnake.com/alfred/2014/08/17/alfred-work-flows.html

+ https://1991421.cn/2019/04/06/b908e228/

  