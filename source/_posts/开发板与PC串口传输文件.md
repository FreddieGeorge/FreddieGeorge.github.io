---
title: 开发板与PC串口传输文件
date: 2023-07-04 11:09:16
tags:
- 嵌入式
- Linux
categories: 
- 嵌入式
excerpt: 使用rz/sz命令在嵌入式Linux和PC之间传输文件
---

## 前言

一直以来我都以为串口这个速率无法传输文件，在与嵌入式通信的时候都是nfs，telnet之类的，这几个月我的USB转网口的模块还坏了，我一直在用sd卡传输文件，不堪其扰。这段时间才看到可以使用串口传输文件，总算是解放了。

> rz(resive z-modem)和sz(send z-modem)是比较古老的工具，而且速率较慢，自己测试传输的时候速率只有`20 kBPS`，只能传输小文件，这个应该是跟波特率有关，能使用nfs的话还是用nfs好一点。

## 参考资料

[韦东山老师的rz/sz教程](https://blog.csdn.net/thisway_diy/article/details/109984270)


## 编译

如果在开发板中没有rz/sz(lrz/lsz)工具，那需要自己编译一个放进开发板中。

```shell
# download source code
wget https://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz

# decompress 
tar -zxvf lrzsz-0.12.20.tar.gz
```

进入解压出来的文件夹中，输入`./configure --prefix=$PWD/__install --host=aarch64-linux-gnu`，根据你的交叉编译工具链修改`--host`后面的内容

然后执行`make && make install`即可在`./__install/bin`目录下看到编译好的工具，将他们放入开发板中即可。

## rz : PC向开发板发送文件

一般PC端的的串口软件都是支持rz发送文件给开发板的，我使用的[MobaXterm](https://mobaxterm.mobatek.net/)。

通过串口连接开发板进入Linux系统之后，输入`rz`，开发板会等待PC端发送文件。

```shell
[root@/]$ rz
▒z waiting to receive.**B0100000023be50
```
 
这个时候对着MobaXterm的终端界面右键(或者`ctrl`+`shift`+右键)就可以打开选择框，选择`Send file using Z-modem`就可以调用资源管理器向开发板传输文件了。

> 我的MobaXterm右键是粘贴内容，所以需要`ctrl`+`shift`+右键才能打开选择框

## sz : 从开发板下载文件到PC

在终端输入`sz <file_to_send>`之后就会进入发送模式等待PC接收文件。

```shell
[root@/]$ sz /root/algo/result.jpg
▒*B00000000000000
```

这时候按照上面的方法打开MobaXterm的选择框，选择`Receive file using Z-modem`即可选择文件保存位置。

## 总结

这个方法只适用于小文件，大文件动辄20多分钟，谨慎使用

最后还是很少用串口传文件，太慢了，目前在使用`adb`，之后有机会出一篇博客。