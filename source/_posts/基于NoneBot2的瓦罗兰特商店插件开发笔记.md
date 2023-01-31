---
title: 基于NoneBot2的瓦罗兰特商店插件开发笔记(未完成)
date: 2023-01-29 18:58:23
tags:
- Nonebot2
- gocqhttp
- python
categories: 
- 开发
- python
excerpt: 我开发瓦罗兰特每日商店qq机器人插件的笔记
hide: true
---

目前进度：刚开始

# 前言

瓦罗兰特，也叫无畏契约，一款拳头的fps游戏。我已经瓦了几个月了，这段时间看到有一些可以查看游戏局内商店的小程序或者discord机器人，而我也有个自己的基于gocqhttp的qq机器人和自己的云服务器，便打算自己也做一个，这篇博客就是我做这个机器人的笔记记录。

# 简单实现

那首先我便去[拳头API官网](https://developer.riotgames.com/apis)尝试寻找也没有官方提供的API接口，找了许久也没找到，我便这github上搜索valorant关键词，发现他们的商店列表都是使用[这个API](https://github.com/HeyM1ke/ValorantClientAPI)，于是我便开始基于这个文档开始编写自己的valorant插件。

> 注意，这个API并非是拳头官方提供，请谨慎使用！！
> 请对自己的拳头账号做好保护措施

## 验证

首先需要使用自己的拳头账号密码获取access_token，user_id，entitlements_toke信息，这些在这个API的[DOC](https://github.com/HeyM1ke/ValorantClientAPI/blob/master/Docs/RSO_AuthFlow.py)中可以获取例程。

运行该例程后发现报错

```shell
aiohttp.client_exceptions.ContentTypeError: 0, message='Attempt to decode JSON with unexpected mimetype: text/html; charset=utf-8'
```

在函数开头加入打印post结果后发现打印信息中有 `[403 Forbidden]` ，很显然这个例程有缺陷，这找帖子发现应该是被 cloudflare 禁止了。
```python
async def run(username, password):
    session = aiohttp.ClientSession()
    data = {
        'client_id': 'play-valorant-web-prod',
        'nonce': '1',
        'redirect_uri': 'https://playvalorant.com/opt_in',
        'response_type': 'token id_token',
    }
    r = await session.post('https://auth.riotgames.com/api/v1/authorization', json=data)
    print(r.text)

    # ... ...
```

在这个github的issue中找到了跟我一样的问题，并找到了一个解决方法，便是使用[别人的轮子](https://github.com/floxay/python-riot-auth),在该项目的README和example中有很详细的使用方法。

> 这个轮子其实应该自己也能实现，但是我对网络相关知识不够熟悉，按照他的样子折腾了一下午还是403，便接受使用轮子了
> 目前先继续import，之后自己再移植到自己的代码内，减少对库的依赖

使用该库之后便可以正确获取access_token等数据

## 获取皮肤

再通过API就可以获取皮肤了，这还是挺简单的，按照github的几个开源代码很容易就弄出来，接下来的问题就是怎么汉化。


