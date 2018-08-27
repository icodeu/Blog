title: CopyOnWrite 浅层次总结
date: 2015-09-14 22:18:44
tags: Java
---

同 ConcurrentHashMap 一样，CopyOnWrite 也是一种多线程并发中使用的容器，实现起来要比 ConcurrentHashMap 简单不少。从 JDK1.5 开始 Java 并发包里提供了两个使用 CopyOnWrite 机制实现的并发容器,它们是 CopyOnWriteArrayList 和 CopyOnWriteArraySet。CopyOnWrite 容器非常有用，可以在非常多的并发场景中使用到，下面就来浅层次总结一下。

<!--more-->

# 源码解析

CopyOnWrite 根据名字，即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以 CopyOnWrite 容器也是一种读写分离的思想，读和写不同的容器。

看一下它的 add 方法就比较清晰了，如下：

```
	public synchronized boolean add(E e) {
		// 新开辟一个长度为原 length + 1 的对象数组
        Object[] newElements = new Object[elements.length + 1];
        // 将原 elements 数组中的内容全部复制到 newElements 数组中
        System.arraycopy(elements, 0, newElements, 0, elements.length);
        // 在 newElements 末尾添加新的元素 e
        newElements[elements.length] = e;
        // 将修改后的数组赋值给原 elements 数组
        elements = newElements;
        return true;
    }
```

另外也需要注意这个方法是同步的，否则多线程写的时候会 Copy 出 N 个副本出来。

下面，至于读方法，真是再简单不过了，如下：

```
	public E get(int index) {
        return (E) elements[index];
    }
```

读的时候不需要加锁，如果读的时候有多个线程正在向 newElements[] 添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的 elements[]。

# 应用场景

CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单。

# 注意问题

- 内存占用问题：因为 CopyOnWrite 的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，那么这个时候很有可能造成频繁的 Young GC 和 Full GC。

- 数据一致性问题：CopyOnWrite 容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果希望写入的的数据马上能读到，就不要使用 CopyOnWrite 容器。




### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)