---
title: "未Root情况下远程进行企业微信打卡或其他打卡操作的两种办法"
draft: false
author: ["nanimonai"]
date: 2024-1-2
hidemeta: true
#description: 2024-1-2   ·  nanimonai
#Toc: false
---

# 为什么研究这个
&emsp;&emsp;缘由是学校的逆天管理制度，企业微信里面创建了一个企业要求所有大学生在21:30-23:00期间进行打卡操作。鉴于企业微信的位置获取不是一般的抽象，除了查看你的GPS信号外还有调用附近的基站进行定位（西瓜和Mike 语），最为逆天的是本人的移动设备是哄蒙系统，未Root，也就无法享用诸多先进科技带来的便利。  

&emsp;&emsp;因此，第一种方法应运而生：  

&emsp;&emsp;1.在手机上安装VMOS Pro，走虚拟机路线。其中的root和Xposed的模块安装均为傻瓜式操作，这里不过多赘述。BTW，虚拟安卓机在大学的另一大用途就是通过学习通考试。  

&emsp;&emsp;2.安装应用变量，Fake location（fuck location不支持虚拟机的安卓版本我记得），而且我们需要的是具有基站模拟功能的位置模拟软件。在当前诈骗猖獗的情况下自建伪基站是不切实且无异于找死的。Fake location需要爆米，我记得7r/月。  

&emsp;&emsp;3.利用自带的Xposed Installer下载，激活应用变量，配置指定应用（如企业微信），其中比较重要的是你的安卓版本，IMEI和MEID这几个需要留意。如果你在之前的手机/母鸡上面打过卡了，这些就要原封不动的抄原来手机。反之则无所谓。  

&emsp;&emsp;4.启动fake location。位置选择到位后启动模拟，同时启动基站模拟。

&emsp;&emsp;5.代价：7r/per month。

&emsp;&emsp;PS：这种方法并非自创，而是由Mike和大西瓜研究而出。此法也有助于不同大学的上课打卡机制，上班族的上班打卡机制。以及Fake location的路线模拟，可以帮助有需要者进行跑步的模拟。（导入运动软件进行操作云云），这方面我不是第一个提出来的，网络上面很多大佬也讲过类似操作,本人这样帮过朋友，证明其可行性。就是画路线的时候别超出操场或者撞墙里面了，以免露馅。

# 好日子的结束

&emsp;&emsp;在一次美美不请假而出游的日子中，我意外发现第一种方法失效了。其具体表现为：地图识别我在A地（GPS），也就是打卡地，但是我的伪装被企业微信识破，即它能准确定位到我的真实所在地B地。而这个方案是在2023.12之前都有效的，且亲自实验过的。  
  
    
&emsp;&emsp;目前原因不明，有思路者欢迎联系我一起探讨。
      
&emsp;&emsp;后经排查原因无果后，高人enadixxoOxoxO指点采用第二种方法。

&emsp;&emsp;OO指出可以使用备用机装企业微信然后写自动脚本完成打卡，并且自己实验成功。  
  
    
&emsp;&emsp;虽然说用的是老旧的备用机。但是仍然要注意自己的安全，隐私等。我最开始试验是在Hamibot上，Hamibot可以远程执行脚本，安全系数低。它算一个云端控制，理论上讲服务器后退可以看我手机的内容甚至控制，所以并不推荐这个平台。里面虽然有日志，但是服务器是外人的。开发文档里面有扫描文件，截屏，OCR获取信息。理论上能获取你手机的任何操作单仅限于显示时候，锁屏没有密码脚本的话，也是开不起来的。如果真要使用也建议是暂时使用几次，不需要使用的时候推荐关闭无障碍。    
  
  
&emsp;&emsp;鄙人的脚本跑在一个叫OpenAuto.js的开源软件上[(Github)](https://github.com/openautojs/openautojs)目前测试也是可行的，甚至优于Hamibot，因为其可以设置脚本的定时执行和循环执行，可以完成很多很多的其他操作而不用把这些内容加到脚本编写里面去。但还是要提供无障碍和一些权限，因此仍然建议找台备用机。    

1.导入脚本    
  
  
2.试运行 DONE.  

&emsp;&emsp;脚本的编写可能是比较难搞定的一个步骤。但是也不难。可以参考[(Github)](https://github.com/hlsky1988/WeChatCheckingIn)，他是写Hamibot上的脚本的，可以参考借鉴思路。   
  
  
&emsp;&emsp;以下是我的打卡脚本，也可以进行一个参考比较，主要是延迟的选定，然后看需不需要增加重新尝试模块，个人感觉没有必要。  
  
  
```
// 确保开启了Auto.js的无障碍服务
auto.waitFor();

// 唤醒并解锁设备（根据你的设备情况，这里需要自定义解锁逻辑）
device.wakeUp();
let { height, width } = device
let x = width / 2
let y1 = (height / 3) * 2
let y2 = height / 3
swipe(x, y1, x + 5, y2, 500)
sleep(3000)
toastLog('启动企业微信,准备打卡')


// 启动企业微信
app.launchPackage("com.tencent.wework");

// 等待企业微信启动
waitForPackage("com.tencent.wework", 5000);

// 等待界面加载的逻辑
sleep(3000);

// 导航到打卡页面的操作
click("工作台");
sleep(1000);
click("打卡");
sleep(1000);

// 执行打卡操作
click("上班打卡");
sleep(1000);

function signAction() {
    toastLog('signAction 开始执行')
    let signIn = text('上班打卡').findOne(1000)
    let signOut = text('下班打卡').findOne(1000)
    if (signIn) {
      let stepLeft = signIn.bounds().left + 10
      let stepTop = signIn.bounds().top + 10
      click(stepLeft, stepTop)
      check()
    } else if (signOut) {
      let stepLeft = signOut.bounds().left + 10
      let stepTop = signOut.bounds().top + 10
      click(stepLeft, stepTop)
      check()
    } else {
      toastLog('打卡未完成,正在检查打卡状态')
      check()
    }
  }

// 可以加上处理打卡后的逻辑，如发送打卡结果通知等，鄙人太懒和垃圾所以没写

// 结束脚本
exit();
```

**DONE.**