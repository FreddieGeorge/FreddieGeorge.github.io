---
title: alacritty配置过程
date: 2023-02-14 15:04:06
tags:
- alacritty
- Linux
categories: 
- Linux
- Manjaro
excerpt: Manjaro配置alacritty的过程
---

## 前言

这几天在跟着视频配置自己的dwm的时候了解到了alacritty，号称是最快的终端模拟器，利用GPU加速使得终端反应速度非常快，而且整体也很简约好看，便打算给自己的Manjaro配置一下。

## 安装

Manjaro安装alacritty也很简单，只需要`sudo pacman -S alacritty`即可。然后在终端输入`alacritty`就可以打开了。

在我安装的时候，alacritty的版本为0.11.0,以下配置都以这个版本为准，如有更新需自行查阅官方文档

## 配置

在我的Manjaro系统中，alacritty的字体间隔会出现问题，按照[这篇博客](https://www.cnblogs.com/siyingcheng/p/11706436.html)的配置文件新建了`~/.config/alacritty/alacritty.yml`文件，在文件中配置如下

```yaml
# 原来tabspaces是8
tabspaces: 4
# 字体使用Soure Code Pro
family: "SauceCodePro"
# 背景不透明度
background_opacity: 0.9
```

配置之后依旧没有变化，只有透明度发生了改变，后来才发现这个font的family字段名称要求很严格，必须要跟字体名字一模一样，可以查看`fc-list`获取字体名字，而且网络上的很多博客和教程都是之前的版本，比如比较多的是0.7.0的版本，很多配置已经不再支持或者变化了，于是我便找到了[官方的配置文件](https://github.com/alacritty/alacritty/blob/master/alacritty.yml),在这里可以看到所有的配置，仔细阅读之后配置如下。

```yaml
# 导入主题颜色配置
import:
    - /home/flork/.config/alacritty/themes/themes/material_theme.yaml

# 设置字体
font:
    normal:
        family: "SauceCodePro Nerd Font Mono"
        style: Regular
    bold:v 
        family: "SauceCodePro Nerd Font Mono"
        style: Bold
    italic:
        family: "SauceCodePro Nerd Font Mono"
        style: Italic

# 设置透明度
window:
    opacity: 0.8

```

配置之后再次启动alacritty即可

至此基本配置完成，alacritty也支持设置快捷键，但是目前还没有这些需求，现在就先弄个基础配置。
