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

这篇博客是自己看dwm源码记录，篇幅应该会比较长，写得详细与粗略都比较看自己的感受，有些可能会展得很开，有些比较不重要的可能就会一笔带过(也可能是我看不懂xD)。同时在看dwm代码的时候我对dwm的认识还不过深刻，如果有错误还请大家帮忙指出。后续了解更多之后我也会回来修正一些错误内容。

2023-02-03:对应x一无所知，很多东西都太抽象了，决定hide这个博客了。先把会的写上去

## dwm源码版本

dwm: version 6.4

## 准备工作

为了方便自己的后续开发和维护，也方便自己后续更换设备的配置迁移，我把dwm的源码放到了自己的github上。为此我新建了一个叫dwm的github仓库。在[上一篇博客](Manjaro从KDE更换为DWM.md)中拉下的dwm代码目录中，先执行 `rm .git -rf`,把suckless.org的git记录删掉，然后再 `git inti`,再把我的dwm仓库添加到远程仓库，最后新建了两个分支 dev 和 suckless_source。

* dev 主要用来后续开发测试用，稳定之后再合并到main中
* suckless_source主要是保存源代码，并在源码中添加自己的注释，此篇博客都是在此分支下工作

## 参考资料

我先是观看了bilibili up主 ["称呼我C先生"的dwm源码解析视频](https://www.bilibili.com/video/BV11U4y1i74t/?spm_id_from=333.788&vd_source=11f15dee056f94ffcf726c3cd64878a3) ，给了我很大帮助，让我不用对着一堆变量名称发懵，也对源码框架有了初步了解。然后就是结合[Xlib的文档](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html)和[dwm的文档](https://dwm.suckless.org/customisation/)一起阅读源码，因为源码中有很多Xlib的库，所以更多是在不断查找Xlib的函数定义和作用。

## 源码分析

### 预备知识

在看源码前，我们先要对dwm的窗口结构有个认识，否则后面很多变量都不知道具体指哪部分。

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

这个是官方的tile *(平铺)*布局，可以看到dwm界面主要部件是tags，title，status，master，stack。

最上面的一行叫状态栏，代码中也叫status

tags也就是dwm界面左上角的1，2，3数字，每个窗口对应一个tag

title和status就跟他们的意思一样。

master和stack就是窗口区域，新的窗口出现在master主窗口上。其他窗口会被推到屏幕右侧的stack上按上下排列堆在一起。快捷键`Alt`+`Enter`可以切换到master或者stack。

### util.c

uitl.c中一共只有两个函数，`die()` 和 `ecalloc()`，都很简单。

 `die()`，顾名思义就是一个打印报错信息并且终止程序的函数

 `ecalloc()`就是一个带报错的calloc，分配内存空间

 同时，在 util.h中定义了三个宏，分别用来返回最大最小值和判断是否在中间

 ### transient.c

 这个是测试Xlib的生成窗口的函数的，并不重要，可以按照文件开头的指令生成一个窗口，也可以自己修改一下，看下创建窗口的流程。

<!-- 操了看不懂, 算了还是有部分看得懂的 -->
 ### drw.c

 这是一个suckless官方利用Xlib封装的一些库，而且函数多为初始化结构体或者修改参数，这边只写几个关键的结构体

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
 | dpy | Display* | Xlib的显示相关数据 |
 | screen | int | 屏幕 |
 | root | XID | 根屏幕 |
 | drawable | XID | **暂时未知** |
 | gc | GC结构体 | Graphics context |
 | scheme | Clr | 指定显示颜色 |
 | fonts | Fnt * | 字体相关 |

```c
typedef struct {
	Cursor cursor;
} Cur;
```
指针相关结构体，在X中，资源都是有一个对应的ID的，类型为unsigned long，跟Drw结构体的root和drawable一样。

```c
typedef struct Fnt {
	Display *dpy;****
	unsigned int h;
	XftFont *xfont;
	FcPattern *pattern;
	struct Fnt *next;
} Fnt;
```
字体链表。

 ### dwm.c

 我们直接从main函数开始看起

```c
int
main(int argc, char *argv[])
{
	if (argc == 2 && !strcmp("-v", argv[1]))
		die("dwm-"VERSION);
	else if (argc != 1)
		die("usage: dwm [-v]");
	if (!setlocale(LC_CTYPE, "") || !XSupportsLocale())
		fputs("warning: no locale support\n", stderr);
	if (!(dpy = XOpenDisplay(NULL)))
		die("dwm: cannot open display");
	checkotherwm();
	setup();
#ifdef __OpenBSD__
	if (pledge("stdio rpath proc exec", NULL) == -1)
		die("pledge");
#endif /* __OpenBSD__ */
	scan();
	run();
	cleanup();
	XCloseDisplay(dpy);
	return EXIT_SUCCESS;
}
```

前两句是很简单的判断参数，输出dwm版本。而下一句`setlocale`是用来[程序本地化](https://blog.51cto.com/u_3078781/3286620)的。

`if (!(dpy = XOpenDisplay(NULL)))`,dpy是一个全局的Display结构体指针，XOpenDisplay函数的作用是打开一个X server的连接并返回Display结构体指针。

`checkotherwm()`则是用于检测是否有其他的wm存在。`setup()`是用来初始化各种全局变量，也不再赘述。

```c
void
scan(void)
{
	unsigned int i, num;
	Window d1, d2, *wins = NULL;
	XWindowAttributes wa;

	if (XQueryTree(dpy, root, &d1, &d2, &wins, &num)) {
		for (i = 0; i < num; i++) {
			if (!XGetWindowAttributes(dpy, wins[i], &wa)
			|| wa.override_redirect || XGetTransientForHint(dpy, wins[i], &d1))
				continue;
			if (wa.map_state == IsViewable || getstate(wins[i]) == IconicState)
				manage(wins[i], &wa);
		}
		for (i = 0; i < num; i++) { /* now the transients */
			if (!XGetWindowAttributes(dpy, wins[i], &wa))
				continue;
			if (XGetTransientForHint(dpy, wins[i], &d1)
			&& (wa.map_state == IsViewable || getstate(wins[i]) == IconicState))
				manage(wins[i], &wa);
		}
		if (wins)
			XFree(wins);
	}
}
```

下一句的`scan()`函数中，主要是遍历窗口。使用了`XQueryTree()`,从xorg的[官方文档](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html#Obtaining_Window_Information)可以看到这个函数是用来获得root窗口的父节点，子节点列表和子节点数量的。并在for循环中遍历子节点，获取子窗口的`XWindowAttributes`信息保存在wa中,这个结构体中就保存了子窗口的x，y，w，h等基本信息，后续的wa.override_redirect则是判断*该子窗口是否覆盖结构控制工具(?)*。如果是true，则wm应该忽略该窗口。`XGetTransientForHint()` 函数则判断子窗口是否为顶级临时窗口，如果是，也忽略。
> 待更新

然后就是最关键的run函数，这个是dwm主要循环。只要判断退出标志running不为0而且XNextEvent在正常等待事件继续则程序会一直进行。如果XNextEvent接受到了事件ev，则进入处理流程。然后就是dwm最关键的 handler函数指针列表。

```c
static void (*handler[LASTEvent]) (XEvent *) = {
	[ButtonPress] = buttonpress,
	[ClientMessage] = clientmessage,
	[ConfigureRequest] = configurerequest,
	[ConfigureNotify] = configurenotify,
	[DestroyNotify] = destroynotify,
	[EnterNotify] = enternotify,
	[Expose] = expose,
	[FocusIn] = focusin,
	[KeyPress] = keypress,
	[MappingNotify] = mappingnotify,
	[MapRequest] = maprequest,
	[MotionNotify] = motionnotify,
	[PropertyNotify] = propertynotify,
	[UnmapNotify] = unmapnotify
};
```

