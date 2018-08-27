title: AndroidStudio 自动生成 SerialVersionUID
date: 2015-12-15 15:04:44 
tags: android
----

关于 SerialVersionUID 的概念和作用就不多说了，自行搜索即可，本篇简单说一下如何在 Android Studio 中自动帮助我们生成(不重复的) SerialVersionUID。

<!--more-->

打开 Settings，切换到 Editor->Inspections->Java->Serialization issues，找到 Serializationzable class without 'serialVersionUID'，将其勾选即可，如图：
![img](http://7xivx9.com1.z0.glb.clouddn.com/androidstudio_serialVersionUID1.jpg)

其实这个意思就是在代码检查时看是否有 serialVersionUID 这个字段，没有就提示你。

之后在实现了 Serializable 接口的类名上打开 Android Studio 的智能提示(智能检查)，就会多出了 Add 'serialVersionUID' field，如图： 
![img](http://7xivx9.com1.z0.glb.clouddn.com/androidstudio_serialVersionUID2.png)


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)