---
layout: post
title: "Ubuntu 下安装好 Wireshark 却找不到 Interface 的解决办法"
date: 2015-06-10 15:25:33 +0800
comments: true
tags: [Linux, Wireshark]
---

今天在做网络攻防时需要在 Ubuntu 中使用 Wireshark 来抓取观察网络数据包，也就有了这篇文章。

<!--more-->

注：本篇文章非原创，部分内容参考：[http://www.94cat.com/blog/?p=1107](http://www.94cat.com/blog/?p=1107)

1.安装 Wireshark
	Ubuntu 下安装 Wireshark 的话直接从`软件中心`搜索安装即可

2.启动 Wireshark
	直接启动 Wireshark 的话会发现 Interface 列表里为空（如果有蓝牙模块，会显示一个蓝牙的），可以使用 `sudo wireshark` 来启动，会有个错误信息，显示 Load Error，但是 Interface 正常，也可以正常使用，但不推荐这种做法。

**3.推荐的方法--使用用户组功能使用 Wireshark**

- 添加用户组，此处以 `wireshark` 用户组为例
```
sudo groupadd wireshark
```

- 将 dumpcap 更改为 wireshark 用户组
```
sudo chgrp wireshark /usr/bin/dumpcap
```

- 让 wireshark 用户组有 root 权限使用 dumpcap
```
sudo chmod 4755 /usr/bin/dumpcap
```
(PS:4754 Wireshark 还是会提示没有权限 )

- 将当前用户加入 wireshark 用户组，我的用户是 `huan`，你添加需要更改这个。
```
sudo gpasswd -a huan wireshark
```

这样就完成了，你可以使用自己的用户打开 Wireshark，能够看到所有 Interface，并且有权限进行操作了。

###个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

###CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

###个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)
