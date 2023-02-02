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

## 版本介绍

dwm: version 6.4

## 准备工作

为了方便自己的后续开发和维护，也方便自己后续更换设备的配置迁移，我把dwm的源码放到了自己的github上。为此我新建了一个叫dwm的github仓库。在[上一篇博客](Manjaro从KDE更换为DWM.md)中拉下的dwm代码目录中，先执行 `rm .git -rf`,把suckless.org的git记录删掉，然后再 `git inti`,再把我的dwm仓库添加到远程仓库，最后新建了两个分支 dev 和 suckless_source。

* dev 主要用来后续开发测试用，稳定之后再合并到main中
* suckless_source主要是保存源代码，并在源码中添加自己的注释，此篇博客都是在此分支下工作

## 参考资料

我先是观看了bilibili up主 ["称呼我C先生"的dwm源码解析视频](https://www.bilibili.com/video/BV11U4y1i74t/?spm_id_from=333.788&vd_source=11f15dee056f94ffcf726c3cd64878a3) ，给了我很大帮助，让我不用对着一堆变量名称发懵，也对源码框架有了初步了解。然后就是结合[Xlib的文档](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html)和[dwm的文档](https://dwm.suckless.org/customisation/)一起阅读源码，因为源码中有很多Xlib的库，所以更多是在不断查找Xlib的函数定义和作用。

<!-- 
## 框架

这篇源码解析按照以下几个方面来写

1. 介绍重要的结构体和变量
2. 
-->


<!-- 不知道要从哪里开始写 -->

