---
title: "lsof 一切皆文件"
categories: ["工具"]
tags: ["lsof"]
date: 2022-03-14
draft: false
---

## 版本
```bash
# lsof -v
lsof version information:
    revision: 4.93.2
    latest revision: https://github.com/lsof-org/lsof
    latest FAQ: https://github.com/lsof-org/lsof/blob/master/00FAQ
    latest (non-formatted) man page: https://github.com/lsof-org/lsof/blob/master/Lsof.8
    ...
```

## 简介
lsof - list open files, 功能为列出所有打开的文件，看着似乎很平常的命令，但在Unix及衍生系统`Everything is a file(descriptor)`一切皆文件的特征加持下，lsof命令也变的异常强大。

一个打开的文件也就可能是普通文件、目录、进程、符号链接、网络文件（如：网络socket、UNIX域socket、NFS文件）等。

```bash
# lsof -p 1 |head -n4
COMMAND PID USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd   1 root  cwd       DIR              252,5     4096          2 /
systemd   1 root  rtd       DIR              252,5     4096          2 /
systemd   1 root  txt       REG              252,5  1620224    7870624 /usr/lib/systemd/systemd
```
上面的输出中：

**COMMAND**: 命令名称，默认只显示前9个字符，可以使用`+c w`进行修改， 例如： lsof +c 10

**PID**: 进程标识号

**TID**: 任务（线程）标识号，空白 TID 列表示一个进程

**USER**: 进程所属用户的用户ID号或登录名

**FD**: 文件描述符编号或描述，包括：
- cwd 当前工作目录
- err FD错误信息
- mem 内存映射文件
- mmap 内存映射设备
- pd 父目录
- rtd 根目录
- txt 程序文本（代码和数据）
- ...

FD后跟以下字符描述打开文件的模式：
- r 读
- w 写
- u 读写
- \- 未知

FD后跟以下字符描述文件的锁形式：
- r 部分文件读锁定
- R 整个文件读锁定
- w 部分文件写锁定
- W 整个文件写锁定
- u 任意长度的读写锁定
- U 未知类型
- ...

**TYPE**: 与文件关联的节点的类型
- REG 普通文件
- DIR 目录
- IPv4 IPv4 socket
- IPv6 IPv6 socket
- inet Internet domain socket
- unix UNIX domain socket
- BLK 设备文件
- CHR 字符文件
- FIFO FIFO队列
- PIPE 管道
- LINK 链接文件
- ...

**DEVICE**: 以逗号分隔的设备编号

**SIZE**, SIZE/OFF or OFFSET: 以bytes为单位的文件大小和/或偏移量。

**NODE**: 本地文件的节点号

**NAME**: 根据上述类型显示的确切名称

## 示例
### 列出所有打开的文件
```bash
lsof
```

### 列出某个进程打开的文件
```bash
lsof -p 1
```

### 列出某个用户打开的文件
```bash
lsof -u ubuntu
```

### 列出进程所有打开的IPv4网络文件
```bash
lsof -i 4 -a -p 1234
```

### 列出打开的IPv6网络文件
```bash
lsof -i 6
```

### 列出和端口相关的文件
```bash
lsof -i :22
lsof -i TCP:1-1024
```

### 禁止将网络号转换为主机名
使用-n参数在网络文件中禁止主机名转换，会更快
```bash
lsof -n
lsof -np 1
```

### 列出TCP或UDP连接
```bash
lsof -i TCP
lsof -i UDP
```

### 列出指定状态下监听的端口
```bash
lsof -i -s TCP:LISTEN
lsof -i -s TCP:ESTABLISHED
```

### 列出连接到某个主机的文件
```bash
lsof -i @192.168.10.10
lsof -i @192.168.10.10:22
```

### 列出某个命令打开的文件
```bash
lsof -c bash
```

### 查看哪些进程打开了文件
```bash
lsof /var/log/message
```

### 查看哪些进程打开了目录及目录下文件
```bash
lsof +d /var/log/
# 使用+D进行目录递归
lsof +D /var/log/
```

## 场景
### 关闭正在运行的连接
当我们会遇到想关闭某个连接又不想重启服务的时候，就可以用lsof查询后使用下面的方法关闭，这种方式也让我们更清晰的了解了文件描述符这种东西。

1.定位本地进程
```bash
ss -nplt 
# ss or netstat
netstat -nplt
```
2.在进程中定位文件描述符
```bash
lsof -np $pid
```
找到连接的匹配文件描述符编号。它将类似于"97u"，记录其中的97

3.连接进程
```bash
gdb -p $pid
```

4.关闭连接
```c
// call close($fileDescriptor) //does not need ; at end.
// 示例
call close(97)
// 退出gdb
quit
```

### 恢复删除的文件
当我们碰到手抖误删了正在使用的文件时，可以先不要慌，使用lsof看一下是否还在占用。
```bash
# 查看文件是否被进程打开
ls -l /var/log/syslog
lsof /var/log/syslog
# 我们使用rm -f /var/log/syslog 模拟文件被删除
rm -f /var/log/syslog
```
1.查看文件是否还在占用
```bash
lsof -n |grep "/var/log/syslog"
# 可以看到类似以下的输出，后面的状态也变为了deleted
rsyslogd  302322  syslog  7w   REG  252,5  349555  32244093 /var/log/syslog (deleted)
...
```

2.找到显示的PID和FD内容
根据前面输出的PID和FD进行定位，查看文件内容
```bash
# tail -f /proc/[PID]/fd/[FD]
tail -f /proc/302322/fd/7
```
3.执行恢复操作
```bash
# 恢复
cat /proc/302322/fd/7 > /var/log/syslog
# 权限修改
chown syslog:adm /var/log/syslog
# 重启占用进程
systemctl restart rsyslog
```

## 参考
- [man lsof](https://man7.org/linux/man-pages/man8/lsof.8.html)
- [Manually closing a port from commandline](https://superuser.com/questions/127863/manually-closing-a-port-from-commandline)