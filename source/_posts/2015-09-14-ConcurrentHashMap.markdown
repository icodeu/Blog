title: ConcurrentHashMap 浅层次总结
date: 2015-09-14 22:18:44
tags: Java
---

学习了 ConcurrentHashMap 的源码，不由地为设计者的思想点赞，现在时间有限，先总结一些我觉得比较有用的知识点，先占个坑，等有时间再详细分析源码。

<!--more-->

# Java 内存模型

## 重排序
- 编译器生成指令的次序，可以不同于源代码所暗示的“显然”版本
- 处理器可以乱序或者并行的执行指令
- 缓存会改变写入提交到主内存的变量的次序

## 内存可见性
- 执行线程 A 的处理器把变量 V 缓存到寄存器中
- 执行线程 A 的处理器把变量 V 缓存到自己的缓存中，但还没有同步刷新到主内存中去
- 执行线程 B 的处理器的缓存中有变量 V 的旧值
- 可以使用 volatile 确保内存可见性

## happens-before 关系

happens-before 关系保证：如果线程 A 与线程 B 满足 happens-before 关系，则线程 A 执行动作的结果对于线程 B 是可见的。如果两个操作未按 happens-before 排序，JVM 将可以对他们任意重排序。

几个与 ConcurrentHashMap 有关的 happens-before 关系法则：
- 程序次序法则：如果在程序中，所有动作 A 出现在动作 B 之前，则线程中的每动作 A 都 happens-before 于该线程中的每一个动作 B
- 监视器锁法则：对一个监视器的解锁 happens-before 于每个后续对同一监视器的加锁
- Volatile 变量法则：对 Volatile 域的写入操作 happens-before 于每个后续对同一 Volatile 的读操作
- 传递性：如果 A happens-before 于 B，且 B happens-before C，则 A happens-before C

# 结构图

## 整体结构图

![ConcurrentHashMap](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/image005.jpg)

## 局部 segment 结构图

![Segment](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/image004.jpg)


# HashEntry 类

```
	// 注意 key、hash、next 都是 final 类型， value 是 volatile 类型
	static final class HashEntry<K,V> { 
        final K key;                // 声明 key 为 final 型
        final int hash;             // 声明 hash 值为 final 型 
        volatile V value;           // 声明 value 为 volatile 型
        final HashEntry<K,V> next;  // 声明 next 为 final 型 
	}
```

# ConcurrentHashMap 类

ConcurrentHashMap 在默认并发级别会创建包含 16 个 Segment 对象的数组。每个 Segment 的成员对象 table 包含若干个散列表的桶。每个桶是由 HashEntry 链接起来的一个链表。如果键能均匀散列，每个 Segment 大约守护整个散列表中桶总数的 1/16。

# 用分离锁实现多个线程间的并发写操作

lock() 只针对某一 segment 进行，最后在 unlock()。

# 用 HashEntery 对象的不变性来降低读操作对加锁的需求

HashEntry 类的 value 域被声明为 Volatile 型，Java 的内存模型可以保证：某个写线程对 value 域的写入马上可以被后续的某个读线程“看”到。在 ConcurrentHashMap 中，不允许用 unll 作为键和值，当读线程读到某个 HashEntry 的 value 域的值为 null 时，便知道产生了冲突——发生了重排序现象，需要加锁后重新读入这个 value 值。这些特性互相配合，使得读线程即使在不加锁状态下，也能正确访问 ConcurrentHashMap。

## 对散列表做非结构性修改

因为使用了 volatile 所以可以保证内存可见性，一个线程修改了某个 value 可以随后被另一个线程看到。

## 对散列表做结构性修改

结构性修改操作包括 put，remove，clear。

（1）clear：将所有的 segment 置空，每个 segment 之前引用的链表还在，只是 segment 不再引用这些链表，所以正在遍历这个链表的读线程依然可以正常执行对该链表的遍历操作。
（2）put：头插法，原有链表并没有修改，也不会影响原来读链表的线程。
（3）remove：
- 执行删除之前的原链表
	
![before-remove](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/image007.jpg)

- 执行删除之后的新链表
![after-remove](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/image008.jpg)

# 用 Volatile 变量协调读写线程间的内存可见性

![volatile](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/image009.jpg)

此处也用到了 happens-before 法则，另外也需要注意 count 这个变量，其被声明为 transient volatile int count; 所有执行写操作的方法（put, remove, clear），在对链表做结构性修改之后，在退出写方法前都会去写这个 count 变量。所有未加锁的读操作（get, contains, containsKey）在读方法中，都会首先去读取这个 count 变量。

# 基于通常情形而优化

由于散列表读操作频率远远大于写操作，所以 ConcurrentHashMap 针对读操作做了大量的优化。通过 HashEntry 对象的不变性和用 volatile 型变量协调线程间的内存可见性，使得大多数时候，读操作不需要加锁就可以正确获得值。这个特性使得 ConcurrentHashMap 的并发性能在分离锁的基础上又有了近一步的提高。

# 高并发性总结

1、减小锁竞争的方式
（1）减小请求同一个锁的频率
（2）减小持有锁的时间

2、ConcurrentHashMap 的高并发性来源于三个方面：
（1）用分离锁实现多个线程间的更深层次的共享访问。
（2）用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
（3）通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

使用分离锁，减小了请求 同一个锁的频率。
通过 HashEntery 对象的不变性及对同一个 Volatile 变量的读 / 写来协调内存可见性，使得 读操作大多数时候不需要加锁就能成功获取到需要的值。由于散列映射表在实际应用中大多数操作都是成功的 读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。
通过减小请求同一个锁的频率和尽量减少持有锁的时间 ，使得 ConcurrentHashMap 的并发性相对于 HashTable 和用同步包装器包装的 HashMap有了质的提高。

[参考文章](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/#ibm-pcon)



### 个人GitHub:  [http://github.com/icodeu](http://github.com/icodeu)

### CSDN博客：[http://blog.csdn.net/icodeyou](http://blog.csdn.net/icodeyou)

### 个人微信号：`qqwanghuan`  技术交流

![image](http://7xivx9.com1.z0.glb.clouddn.com/wxqrcode_260.png)