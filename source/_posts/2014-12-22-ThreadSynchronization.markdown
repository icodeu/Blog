title: Android 中线程同步异步方式总结
date: 2014-10-22 22:18:44 
tags: Android
----

学了这么多年 Java 和 Android，居然连线程同步还是异步都分不清，还好被人问到了『线程同步和异步的区别』，当时我的回答是："这两个有区别吗。。？不都是控制线程之间先后的执行顺序吗？比如 Android 里常用的 Handler、接口回调、事件分发、synchronized、wait、Semaphore、CountDownLatch"等等，我还说了一堆能够控制线程先后执行顺序的方式。后来发现，确实同步和异步不一样，所以本篇文章主要就讲讲同步异步的概念区别以及 Android 中同步异步的各种实现方式。

备注：此篇文章为目录，具体内容后续分篇更新。

<!--more-->

# 同步和异步的区别

首先以一个常见的开发场景来区别一下同步和异步的区别，比如我们要获取一张网络图片并完成显示。在这个场景中我们需要开启两个线程，一个是子线程—即下载图片的线程；另外是主 UI 线程—即图片下载完成后进行显示的线程。针对这个场景分别用两幅实现的流程图来区分同步和异步。



从图中可以看到，二者的区别在于：同步时当前主线程会阻塞，直到子线程通知主线程为止（先不考虑ANR）；而异步的时候主线程可以继续干其它的事情，当子线程完成任务的时候通知一下主线程就可以了，类似于接口回调或消息队列的思想。`所以很关键的一点，在于它们是否会阻塞。`



# Android 中同步的实现方式

1. synchronized、wait/notify 等（Java 中的基础知识，不做过多说明了）
   
2. Semaphore （操作系统中非常重要的概念）
   
3. CountDownLatch
   
4. CyclicBarrier
   
5. 。。。
   
   ​

# Android 中异步的实现方式

1. Handler
2. 接口回调
3. EventBus 等事件分发







### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)