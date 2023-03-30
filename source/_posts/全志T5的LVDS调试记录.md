---
title: 全志T5的LVDS调试记录
date: 2023-03-30 09:43:03
tags:
- T507
- LVDS
categories: 
- 嵌入式
- 全志
excerpt: 这是这段时间的T507调试笔记，坑还是挺多的
hide: true
---

## 前言

这段时间都在忙着全志T507的LVDS调试，最后发现了好多的坑点，虽然最后验证出来是不可行的，但是也还是写一下自己的调试过程和心得吧。

这个调试任务是因为我们需要使用两路lvds输出，一路输出lcd屏，一路需要输出给一个转换芯片，芯片需要输入标准的1080p30，720p30的信号。

LCD的调试倒是简单，对着datasheet调一下前后肩，dclk等就可以了。难点在第二路输出。

## 参考资料 

[全志T5同显](https://m.elecfans.com/article/1810889.html)

[android4.0 A10开发板，如何实现分屏（多屏幕显示）不同的内容](https://www.cnblogs.com/yiguobei99/p/4033919.html)

[A10 LCD调试手册](https://whycan.com/files/201803/A10%20LCD%e8%b0%83%e8%af%95%e6%89%8b%e5%86%8cV1.0.pdf)

[全志同显](https://blog.csdn.net/wangtianxu2008/article/details/79151666)

[全志D1 LCD调试指南](https://whycan.com/files/202108/d1/D1_Linux_LCD_%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97.pdf)

[全志Tina Linux LCD显示屏调试指南](https://cloud.tencent.com/developer/article/2201313)

## 设备树

在全志的sdk中，主要关系到lvds输出的是两个部分，一个是disp，一个是lcd

他们大部分都是有注释的，可以直接查阅注释知道具体意思，也可以看几个参考资料，都写得比较齐全，这次的重点不在这边，便不多赘述。

lcd部分中，需要注意的是`lcd_dclk_freq`，这个是lvds的dclk频率，单位是**MHz**,只能是整数。

从这个表格中我们可以算到，如果要输出1080p30,720p30的图像，则需要74.25MHz和37.125MHz的dclk

>|  制式  | 总扫描线 | 图像区域扫描线 | 水平总像素 | 图像区域水平像素 | 采样频率 |
>| :-: | :-: | :-: | :-: | :-: | :-: |
>|  1080P60  | 1125 | 1080 | 2200 | 1920 | 148.5MHz |
>|  1080P50  | 1125 | 1080 | 2640 | 1920 | 148.5MHz |
>|  720P60  | 750 | 720 | 1650 | 1280 | 74.25MHz |
>|  720P50  | 750 | 720 | 1980 | 1280 | 74.25MHz |

当前驱动无法满足我们的需求。故我们需要更改disp的驱动源码，将`lcd_dclk_freq`的单位改为kHz。

## 更改lcd_dclk_freq单位

具体需要在内核中更改的两个文件，如果你uboot也开了显示，那uboot的对应位置也需要修改。

`drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c`

```c
diff --git a/drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c b/drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c
index 22c5636a1..dc9c87fda 100644
--- a/drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c
+++ b/drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c
@@ -652,15 +652,19 @@ static s32 lcd_clk_config(struct disp_device *lcd)
        disp_al_lcd_get_clk_info(lcd->hwdev_index, &clk_info,
                                 &lcdp->panel_info);
-       dclk_rate = lcdp->panel_info.lcd_dclk_freq * 1000000; 
+    // dclk_rate = lcdp->panel_info.lcd_dclk_freq * 1000000; 
+    dclk_rate = lcdp->panel_info.lcd_dclk_freq * 1000; /* Mhz -> khz */

@@ -722,7 +726,11 @@ static s32 lcd_clk_config(struct disp_device *lcd)
                     pll_rate_set, lcd_rate_set, dclk_rate_set, dsi_rate_set);
        }

+    DE_WRN("disp %d, clk: pll(%ld),clk(%ld),dclk(%ld) dsi_rate(%ld)\n     clk real:pll(%ld),clk(%ld),dclk(%ld) dsi_rate(%ld)\n",
+           lcd->disp, pll_rate, lcd_rate, dclk_rate, dsi_rate,
+           pll_rate_set, lcd_rate_set, dclk_rate_set, dsi_rate_set);
+
     return 0;
 }

 static s32 lcd_clk_enable(struct disp_device *lcd)
@@ -834,8 +842,10 @@ static int lcd_calc_judge_line(struct disp_device *lcd)
                 *               = ht / dclk(Mhz)
                 */
-               lcdp->usec_per_line = panel_info->lcd_ht
+               lcdp->usec_per_line = panel_info->lcd_ht * 1000
						/ panel_info->lcd_dclk_freq;
        }

        if (lcdp->judge_line == 0) {
@@ -1592,9 +1602,13 @@ static s32 disp_lcd_cal_fps(struct disp_device *lcd)
        lcdp->frame_per_sec =
+           DIV_ROUND_CLOSEST(panel_info->lcd_dclk_freq * 1000 *
                                  (panel_info->lcd_interlace + 1),
                              panel_info->lcd_ht * panel_info->lcd_vt);
+       // lcdp->frame_per_sec =
+       //     DIV_ROUND_CLOSEST(panel_info->lcd_dclk_freq * 1000000 *
+       //                        (panel_info->lcd_interlace + 1),
+       //                    panel_info->lcd_ht * panel_info->lcd_vt);

        ret = 0;
 OUT:
@@ -2669,7 +2683,8 @@ static s32 disp_lcd_init(struct disp_device *lcd)
                panel_info = &lcdp->panel_info;

-               timmings->pixel_clk = panel_info->lcd_dclk_freq * 1000;
+               // timmings->pixel_clk = panel_info->lcd_dclk_freq * dclk_scale;
+               timmings->pixel_clk = panel_info->lcd_dclk_freq;
                timmings->x_res = panel_info->lcd_x;
                timmings->y_res = panel_info->lcd_y;
                timmings->hor_total_time = panel_info->lcd_ht;
```

另一个是`drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/tcon/de_lcd.c`

```c
diff --git a/drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/tcon/de_lcd.c b/drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/tcon/de_lcd.c
index 8c811a7ba..604bc5fc3 100644
--- a/drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/tcon/de_lcd.c
+++ b/drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/tcon/de_lcd.c
@@ -663,9 +663,10 @@ static s32 tcon0_cfg_mode_tri(u32 sel, struct disp_panel_para *panel)
 {
        u32 start_delay = 0;
-       u32 de_clk_rate = de_get_clk_rate() / 1000000;
+       // u32 de_clk_rate = de_get_clk_rate() / 1000000;
+       u32 de_clk_rate = de_get_clk_rate() / 1000;
```

这样就可以更改单位为kHz

可以看到我在`lcd_clk_config`函数的末尾增加了一个打印，这个打印原来只会在设置的dclk与实际dclk不符合的时候才会打印，为了方便调试，我在正常情况也添加了这个打印。

然后将`lcd_dclk_freq` 的数值改为 74250 或者 37125，按道理就可以输出了我们需要的时钟了，然而并没有。配置成74250后的打印如下

```c
disp 1, clk: pll(519750000),clk(519750000),dclk(74250000) dsi_rate(519750000)
     clk real:pll(519000000),clk(519000000),dclk(74142857) dsi_rate(0)
```

可以看到实际选择的pll时钟并不对。

## 分频系数

这个pll的值可以在`drivers/clk/sunxi/clk-sun50iw9_tbl.c`中的factor_pllvideo1_tbl中找到，具体用的是哪个结构体得看在对应的clk.dtsi中这个设备挂的是哪个时钟树，比如我当前配置的是lcd1，在`sun50iw9p1-clk.dtsi`中可以看到是video1的pll时钟。

```c 
		clk_tcon_lcd1: tcon_lcd1 {
			#clock-cells = <0>;
			compatible = "allwinner,periph-clock";
			clock-output-names = "tcon_lcd1";
			assigned-clock-parents = <&clk_pll_video1>;
			assigned-clocks = <&clk_tcon_lcd1>;
		};
```

查看该表格，可以找到有很多离散的频率值，那么这个dclk是怎么通过pll算来的呢？

可以看到打印信息中，dclk和pll差了7倍，这个分频系数，在`kernel/linux-4.9/drivers/video/fbdev/sunxi/disp2/disp/de/lowlevel_v33x/disp_al_tcon.c`中可以看到，LVDS的分频系数就是7

```c
	static struct lcd_clk_info clk_tbl[] = {
		{LCD_IF_HV, 16, 1, 1, 0},
		{LCD_IF_CPU, 12, 1, 1, 0},
		{LCD_IF_LVDS, 7, 1, 1, 0},
        /* ... */
```

在`kernel/linux-4.9/drivers/video/fbdev/sunxi/disp2/disp/de/disp_lcd.c`中可以看到函数`lcd_clk_config`,就是实际设置pll和dclk的地方。而当我们设置74250kHz的时候，函数会在pll的table中找到一个最**接近** 74250000 * 7 的值，然后配置为输出时钟。查表可以看到，在分频系数为7的时候，我们得不到准确的时钟。

于是我们便想修改LVDS的分频系数。通过简单计算，修改为分频系数为8，也就是设为`{LCD_IF_LVDS, 8, 1, 1, 0}`。查看打印可以发现软件设置中时钟是准确的。

```c
594000000
disp 1, clk: pll(594000000),clk(594000000),dclk(74250000) dsi_rate(594000000)
     clk real:pll(594000000),clk(594000000),dclk(74250000) dsi_rate(0)
```

事情到这里还没有结束，我用示波器测量LVDS出来的时钟信号，发现时钟信号失真。而可以测量的部分频率也不对，真实值差不多还是pll /7

![LVDS 时钟失真](https://raw.githubusercontent.com/FreddieGeorge/blogImg/main/img/lvds_div_8.jpg)

> 这个配图是37.125MHz的，74.25MHz的没有拍,可以看到测出来的大概是43Mhz，此时的pll是297000000，297Mhz / 7 大概就是42MHz

这个时候我就开始怀疑lvds的时钟分频是否不能修改，但是为了严谨我还是试了很多频率和很多分频系数，发现只要分频系数不为7或者7的倍数，实际的时钟就会失真。而如果是7的倍数，比如14，时钟信号是正常的，但是实际的dclk还是pll / 7。于是我就去翻阅T5的用户手册，找到了这样一段描述

> If interface is LVDS interface, the frequency of DCLK is one seventh of PLL VIDEO, that is, LCDDclk DIV is 7. (For details, see the timming parameter of LVDS)

再查看了LVDS的时序图，可以看到一个时钟周期会发送7个数据，猜测全志的LVDS时序是PLL时钟来采样数据，PLL/7 直接当作dclk频率，达到同步，软件的分频设置并没有用，反而会影响正常的采样。

![LVDS的时序图](https://img-blog.csdn.net/20160128213630063?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 时钟

在分频系数只能为7的情况下，我们只能从时钟那边入手，看下有没有可能找到一个式和的时钟。查看T5的用户手册可以看到PLL时钟的计算公式

> PLL_VIDEO0(4X)= 24 MHz * N/M.
> PLL_VIDEO0(1X)= 24 MHz * N/M/4.

而这个N M的值可以从文档看到是设置在寄存器的，N的取值范围是1-255，M的范围是1-2。

再回到我们看到的那个table，拿出一个举例

`PLLVIDEO1(84,	0,	510000000U)` ，这个84和0，对应的应该就是这两个值，但是可以看到设在寄存器里的值是需要+1才是对应的N和M，故我们验算 24 * 85 / 1 / 4 = 510MHz，验算正确。

那我们就需要考虑一下，根据这个两个公式和取值范围，也没有可能达到我们需要的PLL值，也就是74250000 * 7 = 519750000 和 37125000 * 7 = 259875000。

最后算了一下是不可能的，所以在无法修改分频系数的情况下，得不到我们想要的一个准确时钟，只能获得一个接近值。

## 总结

这次的验证结果就是以全志T507的LVDS时钟设计，是无法满足转换芯片的要求。

> 之前用BT656出来的是可以更改分频系数的，可以达到准确的速率，可是全志T507的BT656把两个LVDS的引脚都复用了

而LCD屏幕的设计，导致它的dclk可以在允许的范围内变化，在一个差不多的dclk内可以正常显示。但是如果要输出给这款转换芯片，那就达不到要求，只能考虑换芯片等方案。

这次的验证花了比较长的时间，但是总的来说还是收获了很多，学会了设备树lcd相关的各个参数的意义，全志disp驱动的大致框架，示波器也熟悉了很多。接下来手头还有几个任务，这段时间的更新可能要继续咕咕咕了。 