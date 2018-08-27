---
layout: post
title: "设置 urllib 的访问超时时间"
date: 2015-07-25 15:25:33 +0800
comments: true
tags: Python
---

```
	import socket
	socket.setdefaulttimeout(5.0)
```

<!--more-->

今天用了 python 的 urllib 库，主要是想编写 sql 基于时间的盲注脚本，要通过网络访问超时来进行异常捕获。`urllib2.openurl(url, timeout=1)` 可以直接设置超时时间，但是 urllib 并不能这样，解决方法：

```
	import socket
	socket.setdefaulttimeout(5.0)
```

因为 urllib 是基于 socket 的，这样就设置了读取网络的超时时间为 5 秒，结合 try 来异常捕获的话，如下：

```
	import socket
	socket.setdefaulttimeout(5.0)

	try:
        urllib.urlopen(url).read()
    except:
        some operation
        pass
```

### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)