title: 蜻蜓FM之 Service 流氓重生
date: 2015-11-11 22:18:44 
tags: Android
----

近来蜻蜓FM事件让中国移动互联网变得更好玩了，本人作为一个程序员，也要全心全意向蜻蜓司的神级程序员好好学习，所以本系列文章就来深刻学习一下蜻蜓FM是如何请来普罗米修斯和宙斯的。

<!--more-->

本篇文章主要来看 Android 中如何能在不同的进程中启动多个 Service 并保持 Service "不被杀死"。



## 创建多个 Service

这个应该和简单，就新建多个 Service 类就好了，在此示例建立了三个 Service，每个代码都是如下：

``` java
public class MyService1 extends Service {
    public MyService1() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

再简单不过的 Service 了，在 Manifest.xml 文件中也注册好，如下：

``` xml
<service
    android:name=".services.MyService1"
    android:enabled="true"
    android:exported="true" >
</service>
```

如此新建好三个 Service 就好了，分别为 MyService1、MyService2、MyService3。另外，为了方便对这三个 Service 进行统一管理，所以我们在新建一个工具类 ServiceUtils，提供一静态方法用于启动 Service，如下：

``` java
// ServiceUtils 工具类用于启动多个Service
public class ServiceUtils {

    public static void startAllService(Context context) {
        context.startService(new Intent(context, MyService1.class));
        context.startService(new Intent(context, MyService2.class));
        context.startService(new Intent(context, MyService3.class));
    }

}
```



接下来我们在 MainActivity 中直接调用该工具类的静态方法直接启动这三个 Service，代码如下：

``` java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		// 直接启动三个 Service
        ServiceUtils.startAllService(this);
    }
}
```

如果你做过 Android 程序，其实以上代码都很简单，下面来实际运行一下，打开模拟器的 `Running app`，如下图：

![initservice](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-initservice.png)



可以看到，一个 QTFM 的进程以及都在该进程中的三个 Service。可是我们再对比一下实际上蜻蜓FM的进程和服务情况，如下图：

![image](http://ww1.sinaimg.cn/large/005SiNxyjw1exvsuo6d8hj30go0pa41w.jpg)

从图中也可以看出来，它的 APP 启动了多个进程，几乎把服务都分散到多个进程中了，这样做的好处就是可以彼此"帮助"，相互唤醒，做到杀不死啊哈哈。其实这个就是 Android 中的多进程编程，我记得之前有个流氓软件防卸载就是使用的这种技术，看来多进程编程在某些程度上被玩坏了。回归正题，我们这里关注的是技术，所以就来看看如何把每个 Service 放在单独的进程中，而不是同一个进程中。

实现多进程编程也不难，打开 Manifest.xml 文件，找到 Service 标签，为 Service 添加 process 属性即可，代码如下：

``` xml
<service
    android:name=".services.MyService1"
    android:enabled="true"
    android:exported="true"
    android:process=":myservice1"></service>
<service
    android:name=".services.MyService2"
    android:enabled="true"
    android:exported="true"
    android:process=":myservice2"></service>
<service
    android:name=".services.MyService3"
    android:enabled="true"
    android:exported="true"
    android:process=":myservice3"></service>
```

给 Service 指定特定的 process 属性就可以了，process 的值可以有两种，其一就是如上写的 `:remote` 以冒号开头的，代表当前程序的私有进程，其二就是不以冒号开头，多以包名开头，代表单独的进程。这里我们先不关心两种方式的具体区别，只先实现启动多个进程。此时我们再运行一下程序，之后切换到设置中的 `Running app` 中查看，如图：

![process](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-process.png)

很明显，目的达到了，每个 Service 都运行在了一个进程中，这个问题我们解决了，继续向下一个问题探索：如何使得 Service 不被杀死？



## 如何使得 Service 不被杀死

此时，我们虽然可以看到确实三个 Service 运行在了三个进程中了，但是如果我们点击某个 Service 中的 "Stop" 按钮，那么该 Service 就被结束掉了，效果如下：



![killservice](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-killservice.gif)



那此时我们需要解决的问题就是，如何在点击了 "Stop" 按钮之后该 Service 不被杀死。实现这个也不难，利用 Service 的生命周期，在其 onDestroy() 中做手脚，几行代码搞定，如下：

``` java
public class MyService1 extends Service {
    public MyService1() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public void onDestroy() {
        // 在这里再次启动 Service
        startService(new Intent(this, MyService1.class));
        super.onDestroy();
    }
}
```

我们给 MyService1 赋予了这种『小强模式』，另外两个 Service 还是和原来一样，这样我们再来分别 Stop 一下，对比看看效果，如下：

![killservice](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-killservice1.gif)



可以看到，每次试图 Stop MyService1 时，它总是又会启动，并且运行时间重新从0开始，而其它两个 Service 都可以轻易被 Stop 掉，形成了明显的对比。所以其实那我们就可以在每个 Service 的 onDestroy() 中调用我们写好的静态工具类的方法来一键启动所有其它 Service 了，不贴代码了，自己尝试下就可以了。



进行到这里，本来我就不想继续写了，因为要探讨的两个问题：『创建多个 Service、如何使得 Service 不被杀死』都已经解决完了，然而真的是这样嘛，我开始也以为是，但是又发现了新的问题，在于某种情况下我点击 Stop 仍然可以杀死服务，你先可以探索一下是否真的这样以及怎样做，然后继续往下看。

———分隔线———

不知道你是否发现了，就是，点击了 Stop 后马上进 Service 再点 Stop，这样就可以杀死该 Service 了，演示如下：

![killservice](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-killservice2.gif)



我非常快的点击了两次该 Service 的 Stop 按钮，惊奇发现所有的 Service 都被杀死了有木有。我们看一下在第二次点击 Stop 之前的界面，貌似和之前的不太一样，如下图：

![stop](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-stop.png)

所以如果这个时候我们立刻点击了 Stop，就将该 App 的所有进程都非正常终结掉了，所以我怀疑有些内存清理或防自启的软件是不是这么做的（仅仅猜测而已，待证实）。



所以目前为止，我们还不能很好的实现服务不被杀死，那我们下面就要继续探索还有什么方式可以帮助我们来自动启动 Service。



## 利用广播来实现自启

这种方法应该早就有了，思想就是静态注册广播接收器，接收一些特定的广播事件，比如开机、连接\断开电源、网络状态变化等来回调 onReceive()，实现具体的自启操作。

我们先定义一个 BroadcastReceiver 用于监听飞行模式的状态变化这一广播事件，Manifest.xml 文件如下：

``` java
<receiver
    android:name=".receivers.MyReceiver"
    android:enabled="true"
    android:exported="true" >
    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
    </intent-filter>
</receiver>
```

添加一 intent-filter 监听飞行模式变化的 action 即可，这个并不需要添加什么额外的权限，如果是监听锁屏开关等则需要相应的权限，据说蜻蜓FM也监听了锁屏。

看下 MyReceiver 的代码，很简单：

``` java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        ServiceUtils.startAllService(context);
    }
}
```

在 onReceive() 中启动需要的 Service 就可以了，这样我们再运行下程序，改变下飞行模式的状态，效果如下图：

![broadcast](http://7xivx9.com1.z0.glb.clouddn.com/qtfm-broadcast.gif)



至此，我们就实现了对一个 app 开启多个进程并分别启动相应的 Service，同时使得 Service 不那么容易被杀死以及收到广播事件后自启。下篇文章将继续研究下一个话题：增加 app 日活。哈哈哈，做个程序员真是越来越有意思。



### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)