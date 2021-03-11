---
title: "tmux的浮动弹窗popup功能"
date: 2021-03-11 00:00:00
tags:
- tmux
---

# 1. 介绍

tmux 已经支持 popup功能, 但是暂时还没有发布到 stable release 版本, 所以需要使用的需要在开发分支上编译使用, 即tmux 版本需要>=3.2

![image-20210311105854535](tmux%E7%9A%84%E6%B5%AE%E5%8A%A8%E5%BC%B9%E7%AA%97popup%E5%8A%9F%E8%83%BD/image-20210311105854535.png)

<!-- more -->

# 2. 使用

### 2.1 升级 tmux 

确认下 tmux 版本, 如果小于3.2就开始升级.

```bash
tmux -V
```

https://github.com/tmux/tmux/releases 下载最新的 release, 编译安装

```bash
./configure && make
sudo make install
```

注意: 升级后如果开启了`tmux-resurrect`,  session会直接卡死

需要把session 杀完后, 并 `tmux kill-server `, 再进一次 (巨坑!!!)

### 2.2 tmux 内使用

弹出一个80%宽高的弹窗

```bash
tmux popup -w 80% -h 80%
```

创建两个别名:


```bash
alias p='tmux popup -w 80% -h 80%' 
alias pp='tmux popup -w 90% -h 90%  "tmux attach -t popup || tmux new -s popup"'
```



# 3. fzf-tmux

在tmux里通过弹窗形式使用 fzf,  参考: https://github.com/kevinhwang91/fzf-tmux-script/

![image-20210311104129324](tmux%E7%9A%84%E6%B5%AE%E5%8A%A8%E5%BC%B9%E7%AA%97popup%E5%8A%9F%E8%83%BD/image-20210311104129324.png)

### 3.1 安装

下面是fzfp脚本,  给个执行权限, 扔到环境变量下, 如 `~/.fzf/bin/`

```shell
#!/usr/bin/env bash

fail() {
    echo "$1" >&2
    exit 2
}

fzf=$(command -v fzf 2>/dev/null) || fzf=$(dirname $0)/fzf
[[ -x "$fzf" ]] || fail 'fzf executable not found'

if [[ -n $TMUX_POPUP_NESTED_FB ]]; then
    eval "$TMUX_POPUP_NESTED_FB" && exec fzf "$@"
fi

tmux -V | awk '{match($0, /[0-9]+\.[0-9]+/, m);exit m[0]<3.2}' || exec fzf "$@"

args=('--no-height')
while (($#)); do
    arg=$1
    case $arg in
    --height | --width)
        eval "${arg:2}=$2"
        shift
        ;;
    --height=* | --width=*)
        eval "${arg:2}"
        ;;
    *)
        args+=("$arg")
        ;;
    esac
    shift
done

opts=$(printf '%q ' "${args[@]}")

[[ -z $height ]] && height=${TMUX_POPUP_HEIGHT:-80%}
[[ -z $width ]] && width=${TMUX_POPUP_WIDTH:-80%}

envs="SHELL=$SHELL"
[[ -n $FZF_DEFAULT_OPTS ]] && envs="$envs FZF_DEFAULT_OPTS=$(printf %q "$FZF_DEFAULT_OPTS")"
[[ -n $FZF_DEFAULT_COMMAND ]] && envs="$envs FZF_DEFAULT_COMMAND=$(printf %q "$FZF_DEFAULT_COMMAND")"

id=$RANDOM
cmd_file="${TMPDIR:-/tmp}/fzf-cmd-file-$id"
pstdin="${TMPDIR:-/tmp}/fzf-pstdin-$id"
pstdout="${TMPDIR:-/tmp}/fzf-pstdout-$id"

clean_cmd="command rm -f $cmd_file $pstdin $pstdout"

cleanup() {
    eval "$clean_cmd"
}
trap 'cleanup' EXIT

mkfifo "$pstdout"

echo -n "trap '$clean_cmd' EXIT SIGINT SIGTERM SIGHUP;" >"$cmd_file"

if [[ -t 0 ]]; then
    echo -n "$fzf $opts > $pstdout" >>"$cmd_file"
else
    mkfifo "$pstdin"
    echo -n "$fzf $opts < $pstdin > $pstdout" >>"$cmd_file"
    cat <&0 >"$pstdin" &
fi
cat "$pstdout" &
tmux popup -d '#{pane_current_path}' -xC -yC -w$width -h$height -E "$envs bash $cmd_file"
```

并且在 zshrc 下 增加启动脚本:

```shell
if [[ -n $TMUX_PANE ]] && (( $+commands[tmux] )) && (( $+commands[fzfp] )); then
    # fallback to normal fzf if current session name is `floating`
    export TMUX_POPUP_NESTED_FB='test $(tmux display -pF "#{==:#S,floating}") == 1'

    export TMUX_POPUP_WIDTH=80%
fi
```

简单创建一个别名:

```bash
alias f="fzfp --preview 'cat {}'"
```

![image-20210311105110624](tmux%E7%9A%84%E6%B5%AE%E5%8A%A8%E5%BC%B9%E7%AA%97popup%E5%8A%9F%E8%83%BD/image-20210311105110624.png)

#  4. tmux-fzf 

通过 fzf 管理 tmux 工作环境, 也可以通过弹窗形式.  参考:  https://github.com/sainnhe/tmux-fzf

### 4.1 安装

+ Add this line to your `~/.tmux.conf`

  ```bash
  set -g @plugin 'sainnhe/tmux-fzf'
  ```

+ Reload configuration, then press prefix + I.

+ To launch tmux-fzf, press prefix + F (Shift+F).

![image-20210311105721312](tmux%E7%9A%84%E6%B5%AE%E5%8A%A8%E5%BC%B9%E7%AA%97popup%E5%8A%9F%E8%83%BD/image-20210311105721312.png)



# 5. 参考资料

+ https://github.com/tmux/tmux/issues/1842
+ https://blog.meain.io/2020/tmux-flating-scratch-terminal/
+ https://github.com/kevinhwang91/fzf-tmux-script
+ https://github.com/sainnhe/tmux-fzf

