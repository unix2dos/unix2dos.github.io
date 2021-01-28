---
title: golang_ide_vscode的使用调试和问题解决
tags: ["golang"]
abbrlink: 127812f2
categories:
  - 1-编程语言
  - golang
date: 2018-05-19 10:41:27
---



### 安装 vscode后的plugins:

1. go
2. vscode-icons
3. code runner
4. markdown preview github
5. markdown auto-open
6. vscode snippets 模板文件: [https://github.com/Microsoft/vscode-go/blob/master/snippets/go.json](https://github.com/Microsoft/vscode-go/blob/master/snippets/go.json)
7. theme molokai 自带
  

<!-- more -->
### vscode增加golang debug调试:

1. xcode-select --install

2. 钥匙链创建证书 dlv-cert
  
3. 证书签名

```    
cd $GOPATH/src/github.com/derekparker
    
git clone https://github.com/derekparker/delve.git  //调试 golang
    
cd delve
    
CERT=dlv-cert make install
```





### 我的vscode配置文件

> setting.json

```
{
    "files.associations": {
        "*.lua.txt": "lua"
    },
    "files.exclude": {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/*.meta": true,
    },
    "files.autoSave": "afterDelay",
    "workbench.colorTheme": "Monokai",
    "workbench.iconTheme": "vscode-icons",
    "workbench.editor.enablePreview": false,
    "editor.fontSize": 14,
    "editor.minimap.enabled": false,
    "editor.formatOnType": true,
    "editor.formatOnSave": true,
    "extensions.autoUpdate": false,
    "extensions.ignoreRecommendations": true,
    "window.zoomLevel": 0,
    "luaide.scriptRoots": [
        "/Users/liuwei/workspace/client3-5/Assets/Resources/Lua"
    ],
    "vim.disableAnnoyingNeovimMessage": true,
    "go.useLanguageServer": true,
    "go.docsTool": "gogetdoc",
    "go.buildOnSave": true,
    "go.lintOnSave": true,
    "go.vetOnSave": true,
    "go.buildFlags": [],
    "go.lintFlags": [],
    "go.vetFlags": [],
    "go.coverOnSave": false,
    "go.useCodeSnippetsOnFunctionSuggest": false,
    "go.formatOnSave": true,
    "go.formatTool": "goreturns",
    "go.goroot": "/usr/local/Cellar/go/1.9.2/libexec",
    "go.gopath": "/Users/liuwei/golang",
}

```

> launch.json

```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "remotePath": "",
            "port": 2345,
            "host": "127.0.0.1",
            "program": "${fileDirname}",
            "env": {},
            "args": [],
            "showLog": true
        }
    ]
}
```






### vscode 遇到的问题

+ flag provided but not defined: -goversion

一个是版本原因, 一个是vscode也要修改配置gopath, 坑爹

>Thank you, I was able to solve this by running brew uninstall --force go and then downloading the latest installer. Anyone who reads this and wants to use brew you could probably just do brew install go after the forced uninstall. I had to restart my terminal and Gogland after doing this.

+ vscode not jump define

```
"go.useLanguageServer": true,
"go.docsTool": "gogetdoc",
```

+ vscode could not launch process: exec: "lldb-server": executable file not found in $PATH

```
xcode-select --install
```


+ vscode jump slow

安装https://github.com/sourcegraph/go-langserver 源码安装 需要 go install

```
"go.useLanguageServer": true,
```


+ vscode output window hide go

~/.vscode/扩展包/package.json 找到显示的

```
"showOutput": "never"
```

