---
title: "Rescue of a Faulty Monitor"
draft: false
author: ["nanimonai"]
date: 2024-11-17
hidemeta: true
description: 低品質のブログ
---
# **失灵显示器拯救记**

​本人在10月份换成了MATX，电脑也从N系笔记本来到了AMD的6750GRE    
  
  
入手一2.5k16寸显示器。本来想自制的，后面听朋友说非常麻烦且容易踩坑，DIY花销不一定比自己小，于是直接购入别人已经做好的所谓A-屏。在我这台的情况是所谓A-屏就是有一个0.1mm不到的色块不显示，个人认为还能接受并且价格也还挺美丽  

​然后问题就来了，小作坊用料就是怪，即这个显示器跑不到预想的分辨率和刷新率。在Windows系统上我的解决办法是使用AMD Software生成一份配置文件，其中他会写一个对于刷新率的感叹号但是你不用去管它。来到Arch我就有些头疼了，他直接给我画面干撕裂了，然后设置界面是经典的“仅支持 640x350”，遂尝试

## 失败的尝试：

GPT师傅建议我去使用cvt创建自定义模式

```1
cvt 2560 1440 240
xrandr --newmode "2560x1600_240.00" 808.75 2560 2792 3072 3584 1440 1443 1448 1493 -hsync +vsync
xrandr --addmode DP-1 "2560x1600_240.00"
xrandr --output DP-1 --mode "2560x1600_240.00"
```
类似这样，一个 RRSetCrtcConfig 错误，表明设置的显示模式参数超出了范围

```1
X Error of failed request:  BadValue (integer parameter out of range for operation)Major opcode of failed request:  140 (RANDR)Minor opcode of failed request:  21 (RRSetCrtcConfig)Value in failed request:  0x78Serial number of failed request:  28
Current serial number in output stream:  28
```


看下日志的 EDID (Extended display identification data) 怎么说

```sudo get-edid | parse-edid
> sudo get-edid | parse-edid
This is read-edid version 3.0.2. Prepare for some fun.
Attempting to use i2c interface
Problem requesting slave address: Device or resource busy
No EDID on bus 1
No EDID on bus 2
No EDID on bus 3
No EDID on bus 4
No EDID on bus 5
No EDID on bus 6
No EDID on bus 7
No EDID on bus 8
No EDID on bus 10
2 potential busses found: 9 11
Will scan through until the first EDID is found.
Pass a bus number as an option to this program to go only for that one.
256-byte EDID successfully retrieved from i2c bus 9
Looks like i2c was successful. Have a good day.
You seem to have too many extension blocks. Will not continue to parse
Something strange happened. Please contact the author,
Matthew Kern at <pyrophobicman@gmail.com>
```



问题：EDID 读取出现异常，显示有太多的扩展块；显示模式创建失败，可能是因为像素时钟超出了硬件支持范围

后面使用gtf创建不同刷新率和分辨率的样式，都不行    

值得一提的是，Linux读取和显示EDID信息是使用read-edid这类或者X.Org Server这样去读取的，而在Windows中则是PowerStrip。而EDID和DDC又是老朋友，通过DDC传输（传奇I2C）

## 成功的尝试：

先创建一个新的 xorg 配置文件，专门处理 EDID 问题

```1
sudo nano /etc/X11/xorg.conf.d/20-amdgpu.conf
```
里面添加：  

```1
Section "Device"
    Identifier "AMD"
    Driver "amdgpu"
    Option "TearFree" "true"
    Option "DRI" "3"
    Option "IgnoreEDID" "false"
    Option "CustomEDID" "DP-1:/etc/X11/edid.bin"
    Option "ModeValidation" "AllowNonEdidModes"
EndSection
```
然后去修改内核参数  

```1
sudo nano /etc/default/grub
```

在 GRUB_CMDLINE_LINUX_DEFAULT 中加入：  
```1
video=DP-1:2560x1600@200 amdgpu.dc=1 amdgpu.dpm=1 amdgpu.modeset=1  
```
这边我看了一眼我的配置文件这行还写着nvidia_drm.modeset=1，顺手给删了（相信暂时应该不会换回N系显卡）

修改后输入如下代码并重启  

```1
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Reboot后看一眼

```1
> xrandr --props
DP-1 connected 2560x1600+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
        RANDR Emulation: 1 
        non-desktop: 0 
                supported: 0, 1
   2560x1600    199.86*+
    ...
```

说明解决了

