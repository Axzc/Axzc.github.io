---
title: hexo 的简单命令与配置
date: 2020-01-14 09:32:06
tags: [tmux, terminal]
categories: 笔记
---

### tmux
tmux 是一款 终端复用工具, 他可以大量的提升工作效率. 比如我经常在 Chrome中的一大堆Tab页中挑啊挑.明明刚打开的页面.这里有个可以回收标签的chrome插件[OneTab](https://chrome.google.com/webstore/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall)
tmux 也是这样它的分页功能可以大量的减少终端窗口.比如在ssh进入到远程服务器时,在不使用tmux的新开的pane中仍然要重新登录.而tmux就不会这样,新开的pane也会保持的ssh连接.


### 安装
在ubuntu 中
`sudo apt-get install tmux`

### 基本概念
输入tmux 命令就相当于开启了一个 服务器, 默认会新建一个会话,然后会话中默认新建一个窗口,窗口中默认新建一个面板
<!--more-->
一个 tmux 会话(Session) 中可以包含多个窗口(window),窗口默认会充满会化界面,一个窗口可以有多个面板(pane)
![](/img/tmux.png)

### 会话
新建回话
```python
tmux  # 创建一个无名称的会话
tmux new -s demo  # 新建一个名称为demo的会话
```
断开当前会话
当会话中操作了一段时间 我希望断开会话下次还能接着用.可以使用 detach命令
```python
tmux detach  # 断开当前会话,会话在后台运行
```
进入之前的会话
使用attach 命令进入到之前的会话, 语法为 tmux  attach-session -t session-name 可简写为tmux a -t session-name 或者tmux a
```python
tmux a  # 默认进入第一个会话
tmux a -t demo  # 进入到名称为demo的会话
```

关闭会话
关闭可以使用 tmux 的kill命令
kill 命令有kill-pane, kill-server, kill-window 和 kill-session 其中Session的语法为 tmux kill-session -t session name
```python
tmux kill-session -t demo  # 关闭demo会话
tmux kill-server  # 关闭服务器, 关闭所有会话
```

查看所有会话
```python
tmux list-session
tmux ls
```

### 快捷键
**系统命令**

前缀 | 命令| 描述
:--:|:---:|:---:
ctrl+b| ?| 显示快捷键帮助文档
ctrl+b| d| 断开当前会话
ctrl+b| D| 选择要断开的会话
ctrl+b| ctrl+z | 挂起当前会话
ctrl+b| r | 强制重载当前会话
ctrl+b| s | 显示会话列表用于选择和切换
ctrl+b| : | 进入命令模式
ctrl+b| [ | 进入复制模式 按q退出
ctrl+b| ]  |  粘贴复制模式中复制的文本
ctrl+b| ~ | 列出提示信息缓存


**窗口(window)命令**

前缀 | 命令| 描述
:--:|:---:|:---:
ctrl+b| c| 新建窗口 
ctrl+b| &| 关闭当前窗口,关闭前需要确认 
ctrl+b| 0~9| 切换到指定窗口 
ctrl+b| p| 切换到上一窗口 
ctrl+b| n| 切换到下一窗口
ctrl+b|  w| 打开窗口列表 用于切换窗口 
ctrl+b|  ,| 重命名当前窗口 
ctrl+b|  .| 修改当前串口编号(适用于窗口重新排序) 
ctrl+b| f|  快速定位到窗口

**面板(pane)命令**

前缀 | 命令| 描述
:--:|:---:|:---:
ctrl+b| "| 当前面板上下一分为二, 下侧为新建面板
ctrl+b| %|当前面板左右一分为二,右侧为新建面板
ctrl+b| x| 关闭当前面板, 关闭前需要确认
ctrl+b| z| 最大化当前面板, 再按一次 后恢复正常
ctrl+b| ;| 切换到最后一次使用的面板  
ctrl+b| {| 向前置换当前面板
ctrl+b| }| 向后置换当前面板 
ctrl+b| t| 显示时钟 
ctrl+b| ctrl+方向键| 调整面板边缘 大小

### 关于 开启256 colors支持
256 colors 真色彩的支持是需要终端支持的,验证终端是否支持真色彩可以在终端里面执行 [24-bit-color.sh](https://github.com/gnachman/iTerm2/blob/master/tests/24-bit-color.sh)
![](/img/24-colors.png)
在 tmux > 2.2 版本之后开始支持真色彩 ,在 ~/.tmux.conf 中添加如下内容
``` python
set -g default-terminal screen-256color
set-option -ga terminal-overrides ",*256col*:Tc" # 这句是关键
```
vim 开启 真色彩
vim 版本 >=7.4.1770 后支持 真色彩在 ~/.vimrc中加入:
``` python
if has("termguicolors")
    " fix bug for vim
    set t_8f=^[[38;2;%lu;%lu;%lum
    set t_8b=^[[48;2;%lu;%lu;%lum

    " enable true color
    set termguicolors
endif
```
最后可以在vim 输入 `:terminal` 进入终端在执行 24-bit-color.sh 来查看是否支持了真色彩