title: Android TextView 上下自带空白边距坑的我好惨
date: 2016-01-23 15:04:44 
tags: Android
----

UI 给了张图，断章取义简单来说就是两个竖直排列的文本之间的距离是10px，我看这个简单啊，直接给下面那个TextView设置一个 marginTop = 10 px(暂时不考虑单位具体用什么)，每个字体的间距我都是这么设计的，结果只要跟文字间距相关的，UI都给我打回来了。我并没觉得自己错了啊，程序里就那么写的怎么会错呢，我还打开了开发者选项去给UI看，UI说她不按开发者选项框出来的红边量，就按上边文本最下面的像素和下边文本最上面的像素量，下面给张图意思一下。如果你也打开开发者选项看TextView的话，你会发现文字上下都有空白的部分，怎么能按照UI的要求，把空白距离也计算到给的尺寸当中呢？我真是研究了好大一半天，现在也把过程和结果写出来供后面踩同样的坑的人参考。

<!--more-->








### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)