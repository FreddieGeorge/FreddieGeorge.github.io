---
title: 全志平台rtl8188fu wifi模块开发
date: 2023-06-26 15:43:32
tags:
- T507
- WiFi
categories: 
- 嵌入式
- 全志
excerpt: 全志平台rtl8188fu模块开发和配置
---

## 前言

好久没更新了，最近的活杂七杂八的，很多还很玄学。这段时间手里有个新板子需要调试，有很多杂七杂八的东西需要配置，主要是这个rtl8188fu比较有意思，这里将调试过程记录下来。


## 参考资料

[全志A64wifi配置](https://blog.csdn.net/qq_41340733/article/details/109072685)

[全志A40i移植 RTL8188FTV/RTL8188FU USB-WiFi](https://blog.csdn.net/weixin_43772810/article/details/121113707)

## 编译驱动源码

我找到的rtl8188fu的驱动源码是[ulli-kroll/rtl8188fu](https://github.com/ulli-kroll/rtl8188fu)，输入以下命令编译

```shell
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- KSRC=/home/flork/T5_SDK/HD250_sdk_test/kernel/linux-4.9/ modules
```

编译报错 `error: label at end of compound statement`,看了下源码报错位置后发现是因为源码中void的函数的lable后面没有内容，类似如下。

```c
void func()
{
    /* ... */
    goto exit;
exit:
}
```

这在不同编译器里面可能会报错，本来想自己慢慢加的，在issue里面随手搜了一下，发现之前有人提过这个issue，并且提交过自己修改好的代码，但是应该是没有合并进去。那我直接拉下[leandropecanhascardua/rtl8188fu](https://github.com/leandropecanhascardua/rtl8188fu/tree/correcting_error_exit_gcc_9_ubuntu)即可直接使用。

至此编译KO文件成功

## 配置

将生成的ko文件拷贝到板子上并安装驱动。

修改`/etc/wpa_supplicant.conf`,默认应该是STA模式，填入路由器的ssid和密码即可

```c
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
ap_scan=1
update_config=1
network={
        ssid="Flork"
        key_mgmt=WPA-PSK
        psk="xxxxxxxxxxx"
        priority=2
}
```

然后执行`wpa_supplicant -Dnl80211 -iwlan0 -B -c /etc/wpa_supplicant.conf`,然后就能通过`ifconfig`看到wlan0的信息了。也可以通过`wpa_cli -i wlan0 status` 查看状态


```c
wlan0     Link encap:Ethernet  HWaddr 48:8F:4C:42:62:05
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

<!-- 此时可以发现还没有连接到网络，需要使用`hostapd -B /etc/hostapd.conf`命令 -->

连接成功后执行`wpa_cli -i wlan0 status`会提示`wpa_state=COMPLETED`

```c
[root@/]$ wpa_cli -i wlan0 status
bssid=64:09:80:18:fe:2f
freq=2442
ssid=Flork
id=0
mode=station
pairwise_cipher=CCMP
group_cipher=TKIP
key_mgmt=WPA2-PSK
wpa_state=COMPLETED
address=48:8f:4c:42:62:05
```

此时仍然不能上网，因为还没分配ip地址，输入`udhcpc -i wlan0 &`,等待路由器dhcp分配ip地址，然后成功ping通网络

```c
[root@/]$ ping www.baidu.com
PING www.baidu.com (14.119.104.189): 56 data bytes
64 bytes from 14.119.104.189: seq=3 ttl=54 time=36.461 ms
64 bytes from 14.119.104.189: seq=4 ttl=54 time=111.119 ms
64 bytes from 14.119.104.189: seq=5 ttl=54 time=83.867 ms
64 bytes from 14.119.104.189: seq=6 ttl=54 time=29.746 ms
64 bytes from 14.119.104.189: seq=7 ttl=54 time=46.891 ms

```

至此wifi模块的基本调试完成
