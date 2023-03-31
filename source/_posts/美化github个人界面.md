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

## 参考样例

Github有个仓库叫[Awesome-Profile-README-templates](https://github.com/kautukkundan/Awesome-Profile-README-templates)，收集了很多漂亮的Github界面。当然很多也没有用到

## 过程 

### 新建profile仓库

首先需要一个仓库，名字与你的github用户名一致，这个仓库官方叫做profile repository。里面的 README.md 的内容就会展示在自己的profile上。

在github中新建一个仓库，命名为`FreddieGeorge`，那么这个就是我的profile repository。在新建仓库的时候勾选上add README.md,即可自动生成一个README.file。后面都是基于这个README.md来操作。

### 添加waka-time代码统计

[waka-time](https://wakatime.com/dashboard)是一个代码时间等统计的插件。我使用的代码编辑软件以VSCode为主，并且很早就安装好了waka-time的统计插件。所以我决定在自己的README中添加代码统计。具体参考了这篇[博客](https://blog.csdn.net/weixin_43233914/article/details/126087735)。

在vscode的配置部分本文就跳过了。

#### 添加Action secret

进入profile仓库中，选择Settings -> Secrets and variables -> Actions -> New repository secret,在Name中填写`WAKATIME_API_KEY`,value就是wakatime的API Key。

然后需要获取Github仓库的API令牌，点击右上角头像-> Settings -> Developer settings -> Personal access tokens -> Tokens(classic),填写好note，设置好过期时间Expiration,然后权限选择repo和user，最后点击Generate token生成令牌，并将生成的令牌复制好

然后就可以再次来到profile仓库中进入新建 repository secret的地方，再新建一个secret，Name设置为`GH_TOKEN`,value就是刚刚复制的令牌。

至此Action secret配置完成。

#### 添加workflows

接下来就需要添加一个让README.md能够每天自动刷新。workflow的文档在[github官网](https://docs.github.com/en/actions/using-workflows/about-workflows)可以找到

workflows的模板大概是

```yaml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

使用的Action仓库是[athul/waka-readme](https://github.com/athul/waka-readme)

