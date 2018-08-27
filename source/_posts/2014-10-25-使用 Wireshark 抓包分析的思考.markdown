---
layout: post
title: "使用 Wireshark 抓包分析的思考"
date: 2014-10-25 15:20:33 +0800
comments: true
tags: [Wireshark]
---

上一篇文章提到了一个问题，即为什么telnet登陆pop邮箱的时候用的是明文，至今不解，为了打消telnet会帮我进行加密传输的小疑问，所以就使用wireshark来看一下数据包的内容。

<!--more-->

依旧使用163邮箱作为实验。打开wireshark，让其抓取我本机网卡的数据包，开始抓包后，通过在cmd中 telnet pop.163.com 110 登陆163服务器，在wireshark里设置过滤规则:  tcp.port==110 (因为pop默认使用110号端口),我每在命令行里输入一个字符，（不用打回车），在wireshark里可以看到数据被封包后发出去了，这也印证了我上篇文章提到过的输入错了按空格是没用的，因为它已经发出去了！我测试输入 `user wanghuan`,`pass wanghuan`，可以明显看到telnet用明文把我输入的内容发送出去了，见下图：

![image](http://img.blog.csdn.net/20141025175729755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWNvZGV5b3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

好了，对于为什么pop使用明文，我还是不明白，若谁能明白，希望能帮我讲解一下。

接下来，我又实验了自己学校的邮箱服务器，通过网页形式登陆，发现传输的依旧是明文，实验过程为：wireshark开始抓包，打开邮箱网页，正常登陆，在wireshark中寻找数据包并设置过滤规则，查看数据包。结果如下图：

![image](http://img.blog.csdn.net/20141025175804153?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWNvZGV5b3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![image](http://img.blog.csdn.net/20141025175821984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWNvZGV5b3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

紧接着，我又测试了foxmail邮件客户端软件，尝试用其登陆我的新浪邮箱，对其进行抓包分析。发现foxmail在往外发数据的时候居然也是明文，结果如下：

![image](http://img.blog.csdn.net/20141025175838036?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWNvZGV5b3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

我顿时就蒙了，是对邮箱服务器本来就可以明文传输，还是我现在实验的对象都对安全问题考虑不周呢？

然后，我选择了新浪邮箱，这个应该还算是国内比较常用的邮箱之一了，如果它都是用明文传输的话，我真是疯了。

所以，赶紧实验，方法如上一样，继续用wireshark，结果如下图所示：

![image](http://img.blog.csdn.net/20141025175856304?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaWNvZGV5b3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这时我发现，我在登陆新浪邮箱的时候，从我网卡里传输出去的数据包是经过`SSL`加密了的（Secure Socket Layer），`不是明文`。其实这才是我预想中的结果，我觉得像用户名密码这种东西在网络上传输必须加密,至少新浪邮箱给了我一点安慰。

wireshark这个东西很强大，但是现在还抓不了局域网中的包，尤其是某些宿舍使用了360安全路由器后。。。
 
###个人github: [http://github.com/icodeu](http://github.com/icodeu)

###CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

###个人微信号：`qqwanghuan`  只为技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)



