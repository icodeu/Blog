title: Android 踩坑奇遇记之 getLocationOnScreen - 获取View在屏幕上的位置
date: 2016-02-27 15:04:44 
tags: Android
----

有个需求，将某个View置于整个屏幕三分之一的地方，Android又不像CSS那样直接有一个 %33 就可以实现，所以获取这个View当前在屏幕中的位置是必不可少的一个步骤，也就是今天要说的 getLocationOnScreen。

<!--more-->

其实这个函数 getLocationOnScreen 应该是见名知意的，就是获取View在屏幕上的位置，让我们也来一睹官方的文档

```文档```

所以简单使用的话代码也很简单，如下

所以location[2]这个数组就保存了当前View相对整个屏幕的绝对坐标

这样一看很简单啊，可是我确遇到了一个问题，用这段代码算出来的不准！我直接说原因了，因为我要测量的View上面还有一个设置了GONE属性的其他View。

先看我的xml文件，可以看到是一个vertical方向的LinearLayout，其中有上下两个view，我要测量的是下面那个view在屏幕上的位置，同时注意上面那个view的visibility="GONE"，所以程序跑起来界面也很简单，如下，我顺便把具体像素值也标注出来了

好了，那么下面咱们就在Java代码里测量一下下面那个view的位置，代码刚才简要说过了，具体如下：

你会发现 Log 中显示的像素值和咱们用 PS 实际测出来的 不一样！Log中打出来的像素值其实是包括了那个Visibility="GONE"的那个view的。

此时如果你再试着把xml中visibility="GONE"的元素给删掉，再运行一遍原来的Java代码，你会发现Log中打出来的和PS实际测量的结果完全一致了。

那么到此，我就在想，既然它上面view已经gone了（注意是gone，而不是visible），为什么还要在screen上占着location，从而导致下面元素的 getLocationOnScreen 结果都不准确，不得其解，看看源码怎么写的吧。

### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)