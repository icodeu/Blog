title: Messenger 的使用及源码解析
date: 2015-10-28 22:18:44 
tags: Android
----

Messenger 和 AIDL 都是 IPC 的方式，并且 Messenger 底层也是基于 AIDL 的，只不过为了使用更方便就封装成了 Messenger。下面依次说一下 Messenger 的基本使用方式以及对 Messenger 源码进行解析。

<!--more-->

# Messenger 简介

Messenger 可以翻译为信使，可以在两个不同的进程中传递 Message 对象，在 Message 中我们又可以放入需要跨进程传递的数据，这样就可以实现 IPC 了。

# Messenger 的使用

我们需要有两个进程，分别为客户端和服务端进程，客户端通过 bindService 的方式来和 Service 建立连接请求并获得 Messenger 对象(Binder 对象)。

### 服务端

1、创建 MessengerService，并在注册时指定 process 属性让其运行在单独的进程中

```
<service
	android:name=".MessengerService"
	android:process=":remote" >
```

2、在 MessengerService 中实现 创建 Messenger、返回 Messenger、处理 Messenger 消息 的逻辑

```
package com.icodeyou.ipcmessenger;
import ...;

public class MessengerService extends Service {
    private static final String TAG = "MessengerService";

    // 用来处理接收 msg 的 handler
    private Handler mMessengerHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            Log.d(TAG, "Recv Data : " + msg.getData().getString("msg"));
        }
    };

    // 定义 Messenger 对象，将关联的 Handler 作为参数传递进去
    private final Messenger mMessenger = new Messenger(mMessengerHandler);

    @Override
    public IBinder onBind(Intent intent) {
    	// 返回 Messenger 中的 Binder 对象
        return mMessenger.getBinder();
    }
}
```

### 客户端

客户端要做的就是 bindService 后在 onServiceConnected() 方法中将 Binder 对象再转换为 Messenger 对象，调用 Messenger#send(msg) 即可向远程发送消息，代码如下：

```
package com.icodeyou.ipcmessenger;
import ....

public class MainActivity extends Activity implements View.OnClickListener {

    private Intent serviceIntent;

    private Messenger mMessenger;

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
        	// 将 Binder 对象转换为 Messenger
            mMessenger = new Messenger(service);
            // 构建个 Message 并将 Bundle 数据设置进去
            Message msg = Message.obtain();
            Bundle bundle = new Bundle();
            bundle.putString("msg", "I am Messenger from Client");
            msg.setData(bundle);
            try {
            	// 使用 Messenger 发送消息给远程服务端
                mMessenger.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        serviceIntent = new Intent(MainActivity.this, MessengerService.class);

        findViewById(R.id.id_btnBind).setOnClickListener(this);
        findViewById(R.id.id_btnUnBind).setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.id_btnBind:
                bindService(serviceIntent, connection, BIND_AUTO_CREATE);
                break;
            case R.id.id_btnUnBind:
                unbindService(connection);
                break;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(connection);
    }
}
```

运行下程序，点击客户端的 Bind 按钮，在 logcat 中查看日志信息，可以看到服务端，即 :remote 收到消息的输出信息，如图：

![messenger](http://7xivx9.com1.z0.glb.clouddn.com/aidl-messenger.png)

以上是客户端远程调用服务端的方法，而服务端还不能主动与客户端进行通信，若要解决这个问题，可通过 Message 的 replyTo 参数传递给服务端，这里就不具体解释了，懂了上面的以后再看下相关代码就明白了。下面咱们来说一下 Messenger 的源码解析。

# Messenger 源码解析

Messenger 是一种轻量级的 IPC 方法，底层仍然是 AIDL，下面来根据代码具体分析。

贴出 Messenger 的关键源代码，分析写在了注释中：

```
package android.os;

public final class Messenger implements Parcelable {
	// IMessenger 是 Binder 对象
    private final IMessenger mTarget;

    // 将 Handler 与 Messenger 关联起来，下面会分析 Handler#getIMessenger()
    public Messenger(Handler target) {
        mTarget = target.getIMessenger();
    }

    // 将服务端的 Binder 转换为客户端的 Binder（AIDL 中使用过）
    public Messenger(IBinder target) {
        mTarget = IMessenger.Stub.asInterface(target);
    }
    
    // 向 Messenger 关联的 Handler 发送一个 Message，会调用该 Handler 的 handlerMessage(msg) 方法
    public void send(Message message) throws RemoteException {
        mTarget.send(message);
    }
    
    // 获取到 Binder 对象
    public IBinder getBinder() {
        return mTarget.asBinder();
    }
}
```

主要就是将能处理消息的 Handler 与 Messenger 相关联，客户端接收到服务端的 Binder 对象后转换为 Messenger 对象，再调用 Messenger#send(msg) 来触发服务端 Handler 的 handleMessage(msg) 方法。

另外，在上述 Messenger 的代码中我们也看到了 handler.getIMessenger()，这个代码实现是怎样的呢，我们可以去看一下 Handler 中相关的代码实现，如下：

```
// Binder 对象
IMessenger mMessenger;

final IMessenger getIMessenger() {
	synchronized (mQueue) {
		if (mMessenger != null) {
			return mMessenger;
		}
		// mMessenger 为空则重新创建
		mMessenger = new MessengerImpl();
		return mMessenger;
    }
}

private final class MessengerImpl extends IMessenger.Stub {
	public void send(Message msg) {
		msg.sendingUid = Binder.getCallingUid();
		// 这里可以看到其实 IMessenger#send() 就是调用了关联的 Handler 的 sendMessage(msg) 方法而已
		Handler.this.sendMessage(msg);
	}
}
```

这样看下来就清晰了很多，知道了底层的实现会对 Messenger 的使用上理解更深入一步。

# Messenger 使用注意事项

- Messenger 一次只处理一个请求，是串行执行的，因此在服务端不用考虑线程同步的问题，这有点类似于 IntentService 中的串行问题，因为都是使用的 Handler、MessageQueue 机制。

- Messenger 只能用来传递消息，不能跨进程调用远程的方法。

- msg.obj 只能传输系统实现了 Parcelable 接口的对象，一般情况下也不要使用 obj 这个字段跨进程传输，可以使用 Bundle 对象来替代 obj，Bundle 可以支持大量的数据类型。


### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)
