title: Android 遇到问题随手记
date: 2014-12-04 15:04:44 
tags: icodeyou
----

从即日起收集自身在 Android 开发过程中遇到的问题及解决方案。

<!--more-->

### 如何添加 .so 库

将 .so 库拷贝到 libs 文件夹，此时不能像 jar 一样右键 add as library，因为根本就没这个菜单项！解决的办法是要借助于 gradle，打开本 module 的 build.grade,添加如下代码即可:

``` java
sourceSets {
	main {
		jniLibs.srcDirs = ['libs']
	}
}
```

### 将应用安装到模拟器上时出现错误 INSTALL_FAILED_CPU_ABI_INCOMPATIBLE

原因是添加 .so 库的时候一般都会添加 armeabi，因为 arm 的手机基本用的都是这个，但是应用在装到模拟器，比如 Genymotion 上的时候就会出现问题了，因为 Genymotion 运行在电脑上用着 x86 或 x64 的处理器，当然就会出现 CPU_ABI_INCOMPATIBLE 的错误了。解决办法是下载 [Genymotion-ARM-Translation.zip](http://7xivx9.com1.z0.glb.clouddn.com/Genymotion-ARM-Translation.zip) 这个文件，然后直接拖到 Genymotion 窗口中等待安装完成就可以了，简直爽。



### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)