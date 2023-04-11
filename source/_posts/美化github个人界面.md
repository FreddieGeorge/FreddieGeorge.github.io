---
title: 美化github个人界面
date: 2023-04-06 15:29:06
tags:
- github
categories: 
- 杂项
excerpt: github个人界面美化记录
---

<!-- 又可以水一篇博客，美滋滋 -->
<!-- 拖更好久了，今天忙完项目填个坑 -->

## 前言

在Github找开源代码的时候，看到了很多大佬漂亮的github界面。于是我决定我也搞一个。于是在周五下午，喝着公司的下午茶把自己的界面捣鼓了一下。~~顺便把这篇博客给水了。~~

示例界面：[FreddieGeorge](https://github.com/FreddieGeorge)

## 参考样例

Github有个仓库叫[Awesome-Profile-README-templates](https://github.com/kautukkundan/Awesome-Profile-README-templates)，收集了很多漂亮的Github界面，根据不同的种类按照文件夹分类了。

## 过程 

### 新建profile仓库

首先需要一个仓库，名字与你的github用户名一致，这个仓库官方叫做profile repository。里面的 README.md 的内容就会展示在自己的profile上。

在github中新建一个仓库，命名为`FreddieGeorge`，那么这个就是我的profile repository。在新建仓库的时候勾选上add README.md,即可自动生成一个README.file。后面的文本需修改都是基于这个README.md来操作。

### 添加waka-time代码统计

[waka-time](https://wakatime.com/dashboard)是一个代码时间等统计的插件。我使用的代码编辑软件以VSCode为主，并且很早就安装好了waka-time的统计插件。所以我决定在自己的README中添加代码统计。具体参考了这篇[博客](https://blog.csdn.net/weixin_43233914/article/details/126087735)。

在vscode的配置部分本文就不再赘述，只需要在插件商店搜索wakatime即可，安装方法网上也很多。接下来只讲github的配置部分

#### 添加Action secret

进入profile仓库中，选择Settings -> Secrets and variables -> Actions -> New repository secret,在Name中填写`WAKATIME_API_KEY`,value就是wakatime的API Key。

然后需要获取Github仓库的API令牌，点击右上角头像-> Settings -> Developer settings -> Personal access tokens -> Tokens(classic),填写好note，设置好过期时间Expiration,然后权限选择repo和user，最后点击Generate token生成令牌，并将生成的令牌复制好

然后就可以再次来到profile仓库中进入新建 repository secret的地方，再新建一个secret，Name设置为`GH_TOKEN`,value就是刚刚复制的令牌。

至此Action secret配置完成。

#### 添加workflows

接下来就需要添加一个让README.md能够每天自动刷新。workflow的文档在[github官网](https://docs.github.com/en/actions/using-workflows/about-workflows)可以找到

workflows的模板大概是

```yaml
name: Waka Readme

on:
  workflow_dispatch: # for manual workflow trigger
  schedule:
    - cron: "0 0 * * *" # runs at every 12AM UTC

jobs:
  update-readme:
    name: WakaReadme DevMetrics
    runs-on: ubuntu-latest
    steps:
      - uses: anmol098/waka-readme-stats@master
        with:
          WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          # 自定义部分
```

使用的Action仓库是[athul/waka-readme](https://github.com/anmol098/waka-readme-stats)。看官方README可以知道不同的flag有不同的含义，我这里挑了比较主要的部分作了一个简单的翻译或者自己的介绍。详细还是以官方README为准


| Flag | 默认 | 含义 |
| :-: | :-: | :-: |
| `LOCALE` | `en` | 使用的语言，键值根据[Short Hand](https://saimana.com/list-of-country-locale-code/)得到 |
| `SECTION_NAME` | `"waka"` | 这是用来寻找何处插入waka统计的匹配字符，后续会提到 |
| `SHOW_UPDATED_DATE` | `"False"` | 是否在图表末尾显示更新时间 |
| `SHOW_LINES_OF_CODE` | `"Flase"` | 是否显示总共写的代码行数 |
| `SHOW_TOTAL_CODE_TIME` | `"True"` | 是否显示总共写的代码时间 |
| `SHOW_PROFILE_VIEWS` | `"True"` | 是否显示profile的浏览量 |
| `SHOW_COMMIT` | `"True"` | 是否显示commit时间段 |
| `SHOW_DAYS_OF_WEEK` | `"True"` | 是否统计在周一到周日各天的commit次数 |
| `SHOW_LANGUAGE` | `"True"` | 是否显示使用语言的时长 |
| `SHOW_OS` | `"True"` | 是否显示使用操作系统时长 |
| `SHOW_PROJECTS` | `"True"` | 是否统计不同项目的工作时长 |
| `SHOW_EDITORS` | `"True"` | 是否显示编辑器 |
| `SYMBOL_VERSION` | `1` | 进度条的填充符号选择 |



我的配置如下
```yaml
name: Waka Readme

on:
  workflow_dispatch: # for manual workflow trigger
  schedule:
    - cron: "0 0 * * *" # runs at every 12AM UTC

jobs:
  update-readme:
    name: WakaReadme DevMetrics
    runs-on: ubuntu-latest
    steps:
      - uses: anmol098/waka-readme-stats@master
        with:
          WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          TIME_RANGE: last_30_days
          SHOW_LINES_OF_CODE: "True"
          SHOW_PROFILE_VIEWS: "False"
          SHOW_COMMIT: "False"
          SHOW_EDITORS: "False"
          SHOW_DAYS_OF_WEEK: "False"
          SHOW_LANGUAGE: "True"
          SHOW_OS: "True"
          SHOW_PROJECTS: "False"
          SHOW_TIMEZONE: "False"
          SHOW_LANGUAGE_PER_REPO: "False"
          SHOW_SHORT_INFO: "False"
          SHOW_LOC_CHART: "False"
          # SYMBOL_VERSION: 1 get graph like  this

          #  ██████████░░░░░░░░░░░░░░░ 
          SYMBOL_VERSION: 2

```

#### README中添加

在README中需要插入waka统计的地方加上两行代码即可，其中`waka`就是`SECTION_NAME`指定的字符

```html
<!--START_SECTION:waka-->
<!--END_SECTION:waka-->
```

### 图标

经常能看到一些profile有各种各样的图标，比如[NoneBot](https://github.com/nonebot/nonebot2)的README开头的一大堆图标。这些都是在[shields.io](https://shields.io/)中能找到，包括语言，下载数等，都可以在这个网站生成。在很多仓库中都可以看到这种图标。