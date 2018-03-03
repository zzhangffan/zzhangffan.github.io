---
title: ThreadLocal分析
date: 2018-02-28 13:00:28
categoreies:
tags: [Java,ThreadLocal]
---
>ThreadLocal 位于java.lang包下，用来为线程提供局部变量。

## ThreadLocal用例 ##
假设我们有一个序列生成器，用于生成从1开始，自增为1的序列，代码如下：
```
public class SeqGenerator{
	private int seq = 0;

	public int generatorSeq(){
		seq += 1;
		return seq;
	}

	public static void main(String[] args){
		SeqGenerator seqGenerator = new SeqGenerator();
		for (int i = 0; i < 3; i++) {//模拟取3次
			System.out.println(Thread.currentThread().getName() + "> " + seqGenerator.generatorSeq());
		}
	}
}
```
程序运行结果如下：
>Thread-0> 1
Thread-0> 2
Thread-0> 3

单线程情况下没有问题，但如果是多线程情况下跑这个程序，代码如下：
```
public class SeqGeneratorThread extends Thread{
	private SeqGenerator seqGenerator;

	public SeqGeneratorThread(SeqGenerator seqGenerator){
		this.seqGenerator = seqGenerator;
	}

	@Override
	public void run(){
		System.out.println(Thread.currentThread().getName() + "> " + seqGenerator.generatorSeq());
	}

	public static void main(String[] args){
		SeqGenerator seqGenerator = new SeqGenerator();
		//创建3个线程，分别获取从1开始的序列
		SeqGeneratorThread seqGeneratorThread1 = new SeqGeneratorThread(seqGenerator);
		SeqGeneratorThread seqGeneratorThread2 = new SeqGeneratorThread(seqGenerator);
		SeqGeneratorThread seqGeneratorThread3 = new SeqGeneratorThread(seqGenerator);
		seqGenerator1.start();
		seqGenerator2.start();
		seqGenerator3.start();
	}
}
```
程序运行结果如下：
>Thread-0> 1
Thread-1> 2
Thread-2> 3

这显然和预期效果不一致，不是说分别获取从1开始的序列吗！可我们看到是多个线程操作结果叠加，这是因为我们多线程操作了同一生成器对象。

ThreadLocal可以解决线程间数据隔离的问题，<note>个人理解：它为每一个使用该变量的线程都保存一份该变量的副本，让线程运行期间都可以独立地操作自己的副本，而不会和其它线程冲突，实现线程间的数据隔离。</note>

使用ThreadLocal改写程序如下：
```
public class SeqGeneratorThread extends Thread{

    private static ThreadLocal<SeqGenerator> threadLocal = new ThreadLocal<SeqGenerator>(){
        @Override
        protected SeqGenerator initialValue(){
            return new SeqGenerator();
        }
    };

    @Override
    public void run(){
        for (int i = 0;i < 3; i++) {
            SeqGenerator seqGenerator = threadLocal.get();
            System.out.println(Thread.currentThread().getName() + "> " + seqGenerator.generatorSeq());
        }
    }

    public static void main(String[] args){
    	SeqGeneratorThread seqGenerator1 = new SeqGeneratorThread();
    	SeqGeneratorThread seqGenerator2 = new SeqGeneratorThread();
    	SeqGeneratorThread seqGenerator3 = new SeqGeneratorThread();
    	seqGenerator1.start();
    	seqGenerator2.start();
    	seqGenerator3.start();
    }
}
```
程序运行结果：
>Thread-0> 1
Thread-1> 1
Thread-2> 1
Thread-1> 2
Thread-0> 2
Thread-1> 3
Thread-2> 2
Thread-0> 3
Thread-2> 3

通过代码我们看出，我们把SeqGenerator交给了ThreadLocal封装。通过get()方法取出SeqGenerator，每个线程操作的都是属于自己的序列生成器。

## ThreadLocal分析 ##

* __成员属性__:
```
//hash值，在类实例创建时被确定
private final int threadLocalHashCode = nextHashCode();

//一个提供原子操作的数字类
private static AtomicInteger nextHashCode =
    new AtomicInteger();

// hash增量
private static final int HASH_INCREMENT = 0x61c88647;
```

* __nextHashCode()__:
```
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

* __set(T value)__:
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
首先获取当前线程的ThreadLocalMap（<note>Thread类中定义了一个ThreadLocalMap类型的实例变量threadLocals</note>），Map不为null时，将ThreadLocal实例作为键和变量副本一起作为方法入参进行set操作。
```
//-----class Thread
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

//-----class ThreadLocal
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

线程中的ThreadLocalMap是懒加载的，当需要存副本的时候才会调用createMap()方法。
```
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

* __get()__：
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```
先获取当前线程的ThreadLocalMap，Map非空的情况下，以ThreadLocal实例为key拿到对应的Entry对象。Entry不为空的情况下返回对应value。

当ThreadLocalMap为null或Entry为null时，调用setinitialValue()方法，该方法如下：
```
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
该方法首先调用initialValue获取初始值，默认返回null，我们可以重写它。
```
protected T initialValue() {
    return null;
}
```
而后的操作与set()基本一致。

* __remove()__：
```
public void remove() {
	ThreadLocalMap m = getMap(Thread.currentThread());
	if (m != null)
		m.remove(this);
}
```
获取当前线程的ThreadLocalMap，Map非空时，将ThreadLocal实例作为入参传入删除。

以上三个方法内部还是调用的ThreadLocalMap的方法。

## ThreadLocalMap分析 ##
在分析ThreadLocalMap之前，先了解几个名词：
* 条目：ThreadLocalMap中保存的Entry实例。
* 陈旧条目：Entry实例还存在，其中保存的key已被GC回收，其value将永远不能被访问，也无法被GC自动回收，这样的Entry称为陈旧条目。
* 插槽、槽：散列地址

ThreadLocalMap使用[哈希表](/20180301/hash-table/)（建议先阅读这篇博文，再看后面的）来实现，内部使用Entry类来存储数据，Entry使用ThreadLocal实例作为key，变量副本作为value。Entry类定义如下：
```
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```
其中存储的对ThreadLocal的引用是一个<note>弱引用</note>。  

* __constructor__:
```
/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```
ThreadLocalMap 有两个构造函数，这里列出了一个，另一个请自行查阅。
1. ` INITIAL_CAPACITY = 16 `
2. ` int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1); `，这里实际上是对 INITIAL_CAPACITY - 1 进行了取余操作。之所以能这样取余是因为 INITIAL_CAPACITY 的值比较特殊，是 2 的 n 次方，减 1 之后低位变为全 1，高位变为全 0。例如 16，减 1 之后对应的二进制为: 00001111，这样其他数字中大于 16 的部分就会被 0 与掉，小于 16 的部分就会保留下来，就相当于取余了。
3. setThreshold()方法设置阈值，` threshold = len * 2 / 3 `

* __set(ThreadLocal key, Object value)__:
```
//设置值
private void set(ThreadLocal key, Object value) {
	
    //ThreadLocalMap使用Entry保存数据，获得保存所有Entry的数组（散列表）
    Entry[] tab = table;
    //表的长度
    int len = tab.length;
    根据ThreadLocal的哈希值计算散列位置
    int i = key.threadLocalHashCode & (len-1);

    //线性探测
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal k = e.get();

        //找到同样的键，直接覆盖值
        if (k == key) {
            e.value = value;
            return;
        }
        //键等于null，替换陈旧条目
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    //映射位置没有元素，创建一个Entry初始化
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //清理一些插槽，如果没有清理，并且数组实际大小大于阈值，调用重新散列的方法
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

* __replaceStaleEntry(ThreadLocal key, Object value, int staleSlot)__:
```
//替换陈旧条目
private void replaceStaleEntry(ThreadLocal key, Object value,
                                       int staleSlot) {
    //取得散列表
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    int slotToExpunge = staleSlot;
    // 优先往前查找要清除的插槽
    for (int i = prevIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = prevIndex(i, len))
        if (e.get() == null)
            slotToExpunge = i;

    // 线性查找
    for (int i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal k = e.get();

        // 找到同样key，交换它们
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            // 未找到其他陈旧条目
            if (slotToExpunge == staleSlot)
            	// 将当前位置记录为要清除的插槽
                slotToExpunge = i;
            // 先expungeStaleEntry()清除要清除的插槽
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

        // 没找到其他陈旧条目
        if (k == null && slotToExpunge == staleSlot)
            // 记录原本陈旧条目位置为要清除的插槽
            slotToExpunge = i;
    }

    // 当前失效key没有被其他条目引用，创建一个条目放入插槽
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    // 当找到其他陈旧条目时，清理陈旧条目
    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}
```

* __expungeStaleEntry(int staleSlot)__:
```
//删除陈旧条目
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 将陈旧条目所在插槽置null
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    // 修改表条目数量
    size--;

    // 重新散列当前槽到下一个空槽间的条目位置
    Entry e;
    // 下一个空槽
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal k = e.get();
        // 陈旧条目直接删除
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
        	// 重新计算散列位置
            int h = k.threadLocalHashCode & (len - 1);
            // 新散列不等于当前位置
            if (h != i) {
            	// 清理当前位置
                tab[i] = null;

                // 新散列位置如果已被占用，往后散列（哈希冲突，开放定址法，线性探测）
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

* __rehash()__:
```
//重新散列
private void rehash() {
	// 清理所有陈旧条目，循环调用了expungeStaleEntry()
    expungeStaleEntries();

    // 扩容策略？当表已被使用阈值3/4时，调整表大小为原来的2倍
    if (size >= threshold - threshold / 4)
    	// 创建一个大小为原来2倍的数组
    	// 重新散列表中所有条目
    	// 重新计算阈值，替换旧表
        resize();
}
```

* __getEntry(ThreadLocal key)__:
```
private Entry getEntry(ThreadLocal key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```
计算关键字的散列位置，根据散列位置取值Entry，找到正确Entry时，直接返回；反之调用直接取值失败的getEntry版本，该方法定义如下：
```
//直接取值失败的getEntry版本
private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;
    //不是空槽
    while (e != null) {
        ThreadLocal k = e.get();
        // 找到条目，返回（出口）
        if (k == key)
            return e;
        //失效条目，直接清除
        if (k == null)
            expungeStaleEntry(i);
        else
        	//线性探测
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

* __remove(ThreadLocal key)__:
```
//删除
private void remove(ThreadLocal key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    //线性探测
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
        	//断开ThreadLocal引用
            e.clear();
            //删除陈旧条目
            expungeStaleEntry(i);
            return;
        }
    }
}
```

## 总结 ##
>1. ThreadLocal作用是提供线程的局部变量。
2. ThreadLocal中的threadLocalHashCode在类实例被创建时被确定，通过支持原子操作的数字类AtomicInteger来获得。
3. 变量副本实际存放在ThreadLocal的静态内部类ThreadLocalMap中，该类也被作为线程Thread类的一个实例属性定义。
4. ThreadLocalMap是采用散列表来实现的，他是用开放定址法来解决哈希冲突。
5. ThreadLocalMap保存的是其静态内部类Entry的实例。通过ThreadLocal的实例属性threadLocalHashCode作为关键字通过一个算法来获得散列地址。
6. Entry类继承了WeakReference类，它对键保持的是一份弱引用。Entry使用ThreadLocal实例作为key，变量副本作为value。
7. <note>cleanSomeSlots方法没看明白</note>


（完）
=====

> 参考博文：[【Java 并发】详解 ThreadLocal](https://www.cnblogs.com/zhangjk1993/archive/2017/03/29/6641745.html)