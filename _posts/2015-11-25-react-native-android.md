---
layout: post
title:  "react-native android配置中的坑"
date:   2015-11-25 15:07:19
tags: [android]
comments: true
---
Nov 25, 2015

简单记录下配置react-native android碰到的坑。

1. 网上的中文版配置都有坑。
2. 一定要仔细看命令行告诉你的话。
3. ANDROID_HOME 要加在本机的环境变量中。有这几个文件都可能是，都写一遍：
   + .zshr
   + .bashrc
   + .bash_profile  
   注意这些都在Macintosh底下
4. 一定要去官网：__[React Native Doc](https://facebook.github.io/react-native/docs/getting-started.html)__

<!--more-->

5. 建议重新下载android sdk manager等工具。在命令行输入：android，下图中的东西必须下下来
   + ![image]({{ site.url }}/img/AndroidSDK1.png)
   + 如果你碰到类似'Could not find method compile() for arguments [com.android.support:appcompat-v7:23.0.1] on org.gradle.api.internal.artifact'的错，一定是没安装全。
   + 有一个官网上没说的，但必须下下来：Google Repository 这个在Extras文件夹中
6. 一开始我找了个快的模拟器Genymotion，可是不能用。因为他的virtul device不能启用GPU HOST。但是Genymotion真的很赞~！！体验真的很赞，真的很赞。重要的话说三遍。速度快配置简单界面友好，我都舍不得卸载。
所以重点是在terminal打上android avd启动虚拟机，然后按下图的配置。
   + ![image]({{ site.url }}/img/CreateAVD.png)
   + 上图是官网的配图。然而那是不对的》《 请仔细看下末尾黄色的警示。你需要install Intel x86 Emulator Accelerator(HAXM)
   
#### 综上所述
　总结就是缺啥补啥。不想一个个搜自己到底缺啥，建议先把Android SDK Manager里能下的都下了，然后再去配置！

<font size="4">我已经迫不及待要开始写几行JS了~！！</font>

![image]({{ site.url }}/img/react_android_success.jpg)

<hr>

后续：

今天终于折腾出了react android切端口这事，不用同时只能运行一个app。现在边iOS模拟器看demo边android 真机改项目，爽～

![image]({{ site.url }}/img/rn_android_ios.jpg)