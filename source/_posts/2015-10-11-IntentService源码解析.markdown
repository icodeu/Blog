title: IntentService 源码解析
date: 2015-10-11 22:18:44 
tags: Android
----

Service 运行在主线程，IntentService 运行在子线程并且可以在执行完 onHandleIntent 方法后自动结束，下面通过分析 Android 源码分析下上述的原理。

<!--more-->

IntentService 代码量不大，下面来看一下主要的变量和几个函数：

```java
public abstract class IntentService extends Service {
	// 保存从 HandlerThread 中得到的 Looper，该 Looper 位于子线程
    private volatile Looper mServiceLooper;
    // 处理 Message 的 Handler，其 Looper 为上面的 mServiceLooper
    private volatile ServiceHandler mServiceHandler;
    private String mName;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
        	// mServiceLooper 取出消息后发送到这里，执行其 onHandleIntent 方法，因为该 Looper 位于子线程，所以下面的两个方法都会运行在子线程
            onHandleIntent((Intent)msg.obj);
            // onHandleIntent 执行完毕后自动结束自己
            stopSelf(msg.arg1);
        }
    }

    /*
     * 只有这个构造方法，所以子类需要明确指定 name，只是方便调试，没有其它用处
     */
    public IntentService(String name) {
        super();
        mName = name;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        // 创建 HandlerThread，其内部会自动创建一个 Looper 对象
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        // 必须调用 start 才可以，因为逻辑写在了线程的 run 方法里
        thread.start();

        // 从 HandlerThread 里获取到其中创建好的 Looper，并注意该 Looper 是在子线程中创建的
        mServiceLooper = thread.getLooper();
        // 用上述 Looper 创建 mServiceHandler
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(Intent intent, int startId) {
    	// Service 启动后直接就向 mServiceHandler 发送消息，则马上就会执行 handleMessage 方法
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    /*
     * 一般不覆盖这个方法
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        // 可暂时忽略 START_REDELIVER_INTENT
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    /*
     * 每次只处理一个任务
     */
    @WorkerThread   // 工作线程，即子线程
    protected abstract void onHandleIntent(Intent intent);
}
```

流程分析：

1、启动后首先执行31行 `onCreate` 方法，在其中创建了 HandlerThread，调用其 start 方法后可在其内部创建一个 Handler，所以此处的 Handler 是在子线程创建的，运行在子线程，这对于后面 onHandleIntent 运行在子线程至关重要。接着使用 thread.getLooper() 获取到了其中的 Looper 对象，并使用该 Looper 当做参数创建了 mServiceHandler。

2、接着是45行的 `onStart` 方法，在此从 mServiceHandler 中获取了一个 Message，其 obj 为传过来的 Intent 对象，并调用 mServiceHandler.sendMessage(msg) 将消息压入消息队列，所以下面就要执行14行的 mServiceHandler 的 `handleMessage()` 方法。

3、进入 handleMessage 方法，第一行就清晰地调用了 `onHandleIntent` 方法，显然就会执行子类覆盖的 onHandleIntent 方法了，并且是在`子线程`，因为 mServiceHandler 中的 Looper 是子线程中创建的。

4、最后调用18行的 stopSelf() 来自我结束。 

以上就分析完了 IntentService 的整个流程，所以可以发现其内部最主要的还是用了 Handler、Looper、Message 通信机制，并且在此需要注意 Looper 是从 HandlerThread 子线程中取出来的，这样才可以让 handleMessage 运行在子线程中。

另外还需要知道的是，如果客户端多次启动同一个 IntentService，则所有任务要串行执行，因为其内部采用 Handler、MessageQueue，Message 要一个一个压进去、一个一个取出来执行，所以就串行了。

关于 HandlerThread，可以看看我写的这篇文章，[HandlerThread 源码解析](http://icodeyou.com/2015/10/11/2015-10-11-HandlerThread%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)
