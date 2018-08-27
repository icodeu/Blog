title: BlockingQueue 浅层次总结
date: 2015-09-15 22:18:44
tags: Java
---

AsyncTask、ThreadPool 中都会用到 BlockingQueue，这是一个接口，它实现的子类有：ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue、SynchronousQueue。本文简要分析总结下 ArrayBlockingQueue。

<!--more-->

### offer(anObject)：

表示如果可能的话，将 anObject 加到 BlockingQueue 里，即如果 BlockingQueue 可以容纳，则返回 true，否则返回 false。

```
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}
```

### offer(anObject, timeout, timeunit):

加不进去就轮询，直到可以加进去为止。

```
public boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(e);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### put(anObject)：
     
把 anObject 加到 BlockingQueue 里，如果 BlockingQueue 没有空间，则调用此方法的线程被阻断直到 BlockingQueue 里有空间再继续。

```
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

### poll(timeout, timeunit)

 取走 BlockingQueue 里排在首位的对象，若不能立即取出，则可以等 timeout 参数规定的时间，取不到时返回null。

```
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

### take()

取走BlockingQueue里排在首位的对象，若BlockingQueue为空，阻断进入等待状态直到BlockingQueue有新的对象被加入为止。

```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

# 四个实现类比较

1、ArrayBlockingQueue：
规定大小的 BlockingQueue，其构造函数必须带一个 int 参数来指明其大小。其所含的对象是以 FIFO（先入先出）顺序排序的。

2、LinkedBlockingQueue：
大小不定的 BlockingQueue，若其构造函数带一个规定大小的参数，生成的 BlockingQueue 有大小限制，若不带大小参数，所生成的 BlockingQueue 的大小由 Integer.MAX_VALUE 来决定。其所含的对象是以 FIFO 顺序排序的。

3、PriorityBlockingQueue：
类似于 LinkedBlockingQueue,但其所含对象的排序不是 FIFO，而是依据对象的自然排序顺序或者是构造函数所带的 Comparator 决定的顺序。

4、SynchronousQueue：
特殊的 BlockingQueue，对其的操作必须是放和取交替完成的。

# 总结

总的来说，BlockingQueue 内部使用的技术有，ReentrantLock，每次存、取前后都要 lock()、unlock()，且默认情况下是 UnfairLock；Condition,具体到 ArrayBlockingQueue 中就是 notEmpty 和 notFull，队列为空或满时无法出队或入队时，就执行 notFull.await() 和 notEmpty.await()；内部的队列是循环队列。




### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)