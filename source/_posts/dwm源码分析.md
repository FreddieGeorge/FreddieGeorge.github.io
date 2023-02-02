---
title: dwm源码分析
date: 2023-02-02 16:02:23
tags:
- dwm
- Manjaro
categories: 
- Linux
- Manjaro
excerpt: 结合文档和资料，粗略解析dwm源码
hide: true
---

# dwm源码分析

## 前言

这篇博客是自己看dwm源码记录，篇幅应该会比较长，写得详细与粗略都比较看自己的感受，有些可能会展得很开，有些比较不重要的可能就会一笔带过(也可能是我看不懂xD),如果有错误还请大家帮忙指出，谢谢。

## 版本介绍

dwm: version 6.4

## 准备工作

为了方便自己的后续开发和维护，也方便自己后续更换设备的配置迁移，我把dwm的源码放到了自己的github上。为此我新建了一个叫dwm的github仓库。在[上一篇博客](Manjaro从KDE更换为DWM.md)中拉下的dwm代码目录中，先执行 `rm .git -rf`,把suckless.org的git记录删掉，然后再 `git inti`,再把我的dwm仓库添加到远程仓库，最后新建了两个分支 dev 和 suckless_source。

* dev 主要用来后续开发测试用，稳定之后再合并到main中
* suckless_source主要是保存源代码，并在源码中添加自己的注释，此篇博客都是在此分支下工作

## 参考资料

我先是观看了bilibili up主 ["称呼我C先生"的dwm源码解析视频](https://www.bilibili.com/video/BV11U4y1i74t/?spm_id_from=333.788&vd_source=11f15dee056f94ffcf726c3cd64878a3) ，给了我很大帮助，让我不用对着一堆变量名称发懵，也对源码框架有了初步了解。然后就是结合[Xlib的文档](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html)和[dwm的文档](https://dwm.suckless.org/customisation/)一起阅读源码，因为源码中有很多Xlib的库，所以更多是在不断查找Xlib的函数定义和作用。

## 源码分析

### 预备知识

在看源码前，我们先要对dwm的窗口结构有个认识，否则后面很多变量都看得不明不白。

从[官方文档](https://dwm.suckless.org/tutorial/)中可以看到窗口的结构是这样的

```text
    +------+----------------------------------+--------+
    | tags | title                            | status |
    +------+---------------------+------------+--------+
    |                            |                     |
    |                            |                     |
    |                            |                     |
    |                            |                     |
    |          master            |        stack        |
    |                            |                     |
    |                            |                     |
    |                            |                     |
    |                            |                     |
    +----------------------------+---------------------+
```

可以看到dwm界面主要部件是tags，title，status，master，stack

### util.c

首先先找个软柿子，uitl.c中一共只有两个函数，`die()` 和 `ecalloc()`，都很简单。

 `die()`是一个打印报错输出并且终止程序的函数

 `ecalloc()`就是一个calloc函数，用来分配空间的，也不再多言

 同时，在 util.h中定义了三个宏，分别用来返回最大最小值和判断是否在中间

 ### transient.c

 这个是测试Xlib的生成窗口的函数的，并不重要，感兴趣的可以按照文件开头的注释生成一个窗口看一下，也可以看下创建窗口的流程。这里面用到的函数和结构体后续都会讲解。


<!-- 操了看不懂, 算了还是有部分看得懂的 -->
 ### drw.c

 这是一个suckless官方利用Xlib封装的一些库，在dwm.c中调用，虽然不会改动他，但是我也顺带一起看了，顺便学习一些Xlib的操作，这边只写几个关键的函数和结构体

 #### Drw结构体

 ```c
typedef struct {
	unsigned int w, h;
	Display *dpy;
	int screen;
	Window root;
	Drawable drawable;
	GC gc;
	Clr *scheme;
	Fnt *fonts;
} Drw;
 ```
 | 成员 | 类型 | 意义 |
 | :--: | :--: | :--: |
 | w | uint | 宽度 |
 | h | uint | 高度 |
 | dpy | Display* | Display结构体指针 |
 | screen | int | 屏幕 |
 | root | 其实就是ulong | 根屏幕 |
 | root | 其实就是ulong | 根屏幕 |

 

 #### drw_create( ) drw_resize( ) drw_resize( )


