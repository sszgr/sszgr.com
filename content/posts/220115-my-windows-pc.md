---
title: "My Windows PC"
date: 2022-01-15
draft: false
---
最近Windows 11的推送越来越频繁了，刚好也想体验下关注已久的[WSLg](https://github.com/microsoft/wslg)项目，就折腾一下自己的备用本，顺便做下配置记录。

Link: https://github.com/sszgr/my-windows-pc

## 操作系统
Microsoft Windows 11

## 目录结构
按常规的方式搞一个D盘作为数据盘，并创建常用的目录结构。创建目录我写了一个小脚本[wt.py](https://github.com/sszgr/my-windows-pc/blob/main/wt.py)，可以配合[example.yaml](https://github.com/sszgr/my-windows-pc/blob/main/example.yaml)生成树结构，方便喜欢这个结构的小伙伴可以直接使用。

```bash
$ ./wt.py -h
usage: wt.py [-h] -f FILE [--mkdir MKDIR] [-t] [-v]

optional arguments:
  -h, --help            show this help message and exit
  -f FILE, --file FILE  configuration file
  --mkdir MKDIR         make directories
  -t, --tree            list contents of configuration in a tree-like format
  -v, --version         show program's version number and exit
```
生成目录
```bash 
$ ./wt.py -f example.yaml --mkdir .
```

结构预览
```bash
$ ./wt.py -f example.yaml -t
example.yaml
├── Work
│   ├── releases
│   ├── project
│   └── packages
├── Tools
│   ├── src
│   ├── pkg
│   └── bin
├── Storage
├── Resource
│   ├── wallpaper
│   └── icons
├── Program Files (x86)
├── Program Files
├── Develpoer
│   ├── gitwork
│   │   └── github.com
│   ├── shwork
│   ├── pywork
│   └── gowork
│       ├── src
│       │   └── golang.org
│       ├── pkg
│       └── bin
└── Downloads
    ├── WeGame
    ├── Thunder
    ├── Firefox
    ├── Edge
    └── Chrome
```

## 终端配置
安装[WSLg](https://github.com/microsoft/wslg)和[WSL](https://docs.microsoft.com/en-us/windows/wsl/install)
```sh
wsl --update
wsl --list --online

# 安装一个Ubuntu用于Linux下开发，配合VSCode使用
wsl --install -d Ubuntu-20.04
wsl --set-version Ubuntu-20.04 1

# 安装一个Kali方便工具的文档查看，正常测试还是使用虚拟机
wsl --install -d kali-linux
wsl --set-version kali-linux 2
```
安装[Windows Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701)并设置为默认终端

将Ubuntu-20.04配置为默认启动，主题配置为`Solarized Light`，启动目录设置为家目录
```json
{
    "name": "Ubuntu-20.04",
    "colorScheme": "Solarized Light",
    "startingDirectory": "\\\\wsl$\\Ubuntu-20.04\\home\\st"
    ...
}
```

## 常用工具
工具会安装在`Program Files`或`Program Files (x86)`的目录下，一般只需要调整安装前缀。

1. [Google Chrome](https://www.google.com/chrome)

2. [Firefox](https://www.mozilla.org/)

3. [Python](https://www.python.org/)

4. [Visual Studio Code](https://code.visualstudio.com/)
    
5. 迅雷极速版

6. [Process Hacker](https://processhacker.sourceforge.io/) 

7. [HeidiSQL](https://www.heidisql.com/)

8. [Cutter](https://cutter.re/)

9.  [Procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)

10. [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/)

11. [WinSCP](https://winscp.net/)

12. [GifCam](https://blog.bahraniapps.com/gifcam/)

13. [Wireshark](https://www.wireshark.org/)

14. [CherryTree](https://www.giuspen.com/cherrytree/)

15. [VLC media player](https://www.videolan.org/)

16. [USBWriter](https://sourceforge.net/projects/usbwriter/)

17. [WeGame](https://www.wegame.com.cn/)

18. [Microsoft Network Monitor](https://www.microsoft.com/en-us/download/4865)

19. [TickTick](https://ticktick.com/)

20. [腾讯会议](https://meeting.tencent.com/)

21. [Zoom](https://zoom.us/)

22. [Burp Suite](https://portswigger.net/burp)

23. [Kleopatra](https://www.gpg4win.org/)
