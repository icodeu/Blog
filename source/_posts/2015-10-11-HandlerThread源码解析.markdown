title: HandlerThread 源码解析
date: 2015-10-11 22:18:44 
tags: Android
----

Android 为我们提供好了一个类——HandlerThread，在其内部维护了一个 Looper 对象，通过线程的 start 方法启动后即可创建内部的 Looper 对象，并且此 Looper 对象运行在子线程，我们还可以通过 handlerThread.getLooper() 来取到这个 Looper 对象，免得我们自己创建了。

<!--more-->

根据源码分析下 HandlerThread 的机制，下面是关键代码：

```java
/*
 * 开启一个内部有 Looper 的线程，这个 Looper 可以用来创建其它 Handler，注意需要调用 start() 方法
 */
public class HandlerThread extends Thread {
    // 线程优先级
    int mPriority;
    // 线程 id
    int mTid = -1;
    // 内部维护的 Looper 对象
    Looper mLooper;

    public HandlerThread(String name) {
        super(name);
        // 指定默认的线程优先级
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
    
    /*
     * 可以覆写该回调方法来在 Looper.loop() 前执行一些操作
     */
    protected void onLooperPrepared() {
    }

    /*
     * 关键方法，注意需要调用 start 方法
     */
    @Override
    public void run() {
        mTid = Process.myTid();
        // 创建 Looper 对象，此处用到了 ThreadLocal
        Looper.prepare();
        synchronized (this) {
            // 获取到 Looper 对象
            mLooper = Looper.myLooper();
            // 此处与 getLooper() 中的 wait 相对应
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        // loop 之前的操作，默认为空
        onLooperPrepared();
        Looper.loop();
        // 为什么要为 -1？
        mTid = -1;
    }
    
    /*
     * 关键方法，返回该线程中维护的 Looper. 如果此线程没有被 start 或者其它原因导致 isAlive() 的值是false，则此方法返回 null
     * 如果此线程已经被 start 了，那么这个方法将会阻塞直到线程中的 Looper 已经被初始化了
     */
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    // 阻塞
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    /*
     * 退出 HandlerThread 中的 Looper
     * 直接退出 Looper，即使 MessageQueue 中还有未处理的消息，所以此种退出方法不安全
     * 退出后，handler.sendMessage(msg) 将返回 false
     */
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    /*
     * 处理完 MessageQueue 中的所有消息后才退出 Looper，所以此种退出方法比较安全
     * 退出后，handler.sendMessage(msg) 将返回 false
     */
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    /*
     * 返回线程 ID
     */
    public int getThreadId() {
        return mTid;
    }
}
```

学习完了 HandlerThread，那么思考个问题，一个 Looper 可以对应多个 Handler 吗？具体答案见 [说说 Looper#getMainLooper](http://icodeyou.com/2015/10/11/2015-10-11-getMainLooper/)。

关于 HandlerThread 的具体应用，请看 [IntentService 源码解析](http://icodeyou.com/2015/10/11/2015-10-11-IntentService%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)。


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)
