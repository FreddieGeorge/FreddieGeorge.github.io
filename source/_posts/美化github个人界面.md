---
title: 美化github个人界面
date: 2023-03-10 15:29:06
tags:
- github
categories: 
- 杂项
excerpt: github个人界面美化记录
hide: true
---

<!-- 又可以水一篇博客，美滋滋 -->

## 前言

在Github找开源代码的时候，看到了很多大佬漂亮的github界面。于是我决定我也搞一个。于是在周五下午，喝着公司的下午茶把自己的界面捣鼓了一下。~~顺便把这篇博客给水了。~~

示例界面：[FreddieGeorge](https://github.com/FreddieGeorge)

## 参考模板

Github有个仓库叫[Awesome-Profile-README-templates](https://github.com/kautukkundan/Awesome-Profile-README-templates)，收集了很多漂亮的Github界面。当然很多也没有用到

## 过程 

### profile 界面展示README

首先需要一个仓库，名字与你的github用户名一致，官方叫做profile repository。里面的 README.md 的内容就会展示在自己的profile上。

### 添加waka-time代码统计

waka-time是一个代码时间等统计的插件。我使用的代码编辑软件以VSCode为主，并且很早就安装好了waka-time的统计插件。所以我决定在自己的README中添加代码统计。具体参考了这篇[博客](https://blog.csdn.net/weixin_43233914/article/details/126087735)。

#### 添加secret

在你的profile repository中的，也就是与你的用户名同名的仓库中