title: mac工具--通过 arp 欺骗抓取局域网内设备数据包
date: 2015-08-23 22:18:44
tags: Mac
---


使用 `Debookee` 抓取同局域网内设备数据包，太神奇了。

<!--more-->

因为有个需求，就是想看某手机app内部网络部分是如何实现的，所以要抓取其数据包（主要是 Http 协议部分），Windows 下可以用 Fidder 为手机设置代理实现，Mac 下有一款 `Debookee` 的软件可以实现同样的功能，但是原理不同 Fidder 设置代理。首先先看截图：

这是扫出来的局域网内所有主机情况：

![扫描主机列表](http://7xivx9.com1.z0.glb.clouddn.com/debookeedebookee_list.png)

将目标主机设置为 target，此处是我的手机，然后再点击左上角的 `Start Dbk`，就会抓取到手机上的数据包，如图：

![抓手机数据包](http://7xivx9.com1.z0.glb.clouddn.com/debookeedebookee_phone.png)

很神奇，什么都不用设置，只要在 `Debookee` 上点几下，同个局域网中所有设备数据包我都能监控到了。基于什么原理呢？

我的另一台 Samsung 电脑打开了 Wireshark 抓取数据包，发现 `Debookee` 只要一扫描或者对其抓包，就会有满屏幕的 ARP 欺骗--所以原理也就清楚了，是`基于 ARP 欺骗`的（感谢大哥教导），如图：

![ARP 欺骗](http://7xivx9.com1.z0.glb.clouddn.com/debookeedebookee_arp.jpg)

其实很简单，最后，视频演示下吧：

<div class="video-container">
	<video src="http://7xivx9.com1.z0.glb.clouddn.com/debookeeDebookee.mp4" controls="controls"></video>
</div>

总的来说，这款工具非常好用！—— `Debookee`

另外，还有另一种方式，即通过`设置中间代理`的方式来抓包，在我的这篇文章中有讲解 [Charles 的安装破解与使用](http://icodeyou.com/2015/08/30/2015-08-30-%20mac%E4%B8%8B%E6%8A%93%E5%8C%85%E5%B7%A5%E5%85%B7--Charles/)

### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)