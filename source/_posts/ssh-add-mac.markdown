title: Mac 上 ssh-add 永久将私钥添加到 Keychain
date: 2016-01-17 15:04:44 
tags: Git
----

两种连接 Git 服务器的方式，分别为 HTTPS 和 SSH，显然更推荐后者，所以我们经常使用命令 `ssh-keygen -t rsa -C “me@icodeyou.com”` 来生成 SSH 的公钥和私钥，之后执行 `ssh-add privateKey` 将 SSH 的私钥添加进去，但是发现了一个问题就是`每次重启电脑后都需要重新 ssh-add`，显然每次重启后都需要重新添加让我等程序员肯定受不了，解决办法就是在添加 ssh 私钥的时候使用如下命令： `ssh-add -K privateKey`，即可一劳永逸将私钥添加进 Mac 本身的钥匙串中，即 Keychain。下面简单解释下原理。

<!--more-->

首先得了解一件事：ssh-add 这个命令不是用来永久性的记住你所使用的私钥的。实际上，它的作用只是把你指定的私钥添加到 ssh-agent 所管理的一个 session 当中。而 ssh-agent 是一个用于存储私钥的临时性的 session 服务，也就是说当你重启之后，ssh-agent 服务也就重置了，session 会话也就失效了。

既然 ssh-agent 是个临时的，那么对于 Mac 来说，哪里可以永久存储的，显然就是 Keychain 了，在执行 `ssh-add -K privateKey` 后可以打开偏好设置中的 Keychain来观察一下前后的变化，是不是多出了 SSH 的条目，见下图。

![ssh-add-K](http://7xivx9.com1.z0.glb.clouddn.com/ssh-add-K.png)

后记：其实我最开始以为是 zsh 的问题，看来我是冤枉我的好兄弟了。。。


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)