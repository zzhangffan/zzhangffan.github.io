---
title: ReentrantLock解析
date: 2018-03-08 09:22:30
categoreies:
tags: [Java,Concurrency]
---

## 示例 ##
在ReentrantLock中，调用 lock()方法获取锁；调用unlock()方法释放锁。
```
class ReentrantLockExample{
	int a = 0;
	ReentrantLock lock = new ReentrantLock();

	public void writer(){
		lock.lock();
		try{
			a++;
		} finally{
			lock.unlock();
		}
	}

	public void reader(){
		lock.lock();
		try{
			int i = a;
			System.out.println("reader i = " + i);
		} finally{
			lock.unlock();
		}
	}
}
```

## 解析 ##
ReentrantLock 拥有两个构造函数：
```
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
ReentrantLock的实现依赖于 java 同步器框架 AbstractQueuedSynchronizer(AQS)。 
* AQS 使用一个整型的volatile变量（state）来维护同步状态。
```
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    	...

    /**
     * The synchronization state.
     */
    private volatile int state;

    	...
}
```

* AQS 使用一个内部类（Node）来实现 CLH lock queue 的变种。
每个节点中除了存储当前线程，前后节点引用，还有一个waitStatus变量，用于描述节点当前的状态。
waitStatus取值有：
1. CANCELLED（1） 取消状态
2. SIGNAL（-1） 等待触发状态
3. CONDITION（-2） 等待条件状态
4. PROPAGATE（-3） 状态需要向后传播
5. 0 初始化


ReentrantLock 类图（部分）：
![ReentrantLock-UML](/images/ReentrantLock/ReentrantLock-UML.png)
> 图片来自：深入理解JAVA内存模型一书

ReentrantLock 分为公平锁和非公平锁，先分析公平锁。
使用公平锁，调用加锁方法 lock() 时，方法调用轨迹：
1.ReentarntLock ： lock()
2.FairSync : lock()
3.AbstractQueuedSynchronizer : acquier(int arg)
```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
4.FairSync : tryAcquire(int acquires)  //尝试加锁（钩子方法）
tryAcquire 源码如下：
```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    //获取同步状态
    int c = getState();
    if (c == 0) {
    	//1.前面是否有排队的
    	//2.比较和设置状态
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            //设置拥有独占访问权限的线程为自身
            setExclusiveOwnerThread(current);
            //获锁成功
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { //重入
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
5.tryAcquire 失败入等待队列：
5.1 AbstractQueuedSynchronizer : addWaiter(Node mode) 添加等待成员
```
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    //等待队列非空，排队当前为队尾
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //空队列，初始化后排队
    enq(node);
    return node;
}
```
5.2 AbstractQueuedSynchronizer ： acquireQueued(final Node node, int arg) 自旋获取锁
```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
        	//获取前驱节点
            final Node p = node.predecessor();
            //前驱节点是头节点，并且获取成功获取状态
            if (p == head && tryAcquire(arg)) {
            	//当前节点为头节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获取失败
            //1.判断是否需要阻塞线程
            //2.阻塞和检查中断（？）
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                //记录线程中断状态
                interrupted = true;
        }
    } finally {
        if (failed)
        	//取消获取
            cancelAcquire(node);
    }
}
```
5.3 AbstractQueuedSynchronizer：shouldParkAfterFailedAcquire(Node pred, Node node) //判断是否需要阻塞线程
```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        //前一个是等待状态。直接设为等待
        return true;
    if (ws > 0) {
        //跳过所有被取消的节点
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        //修改前驱为等待状态
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
5.4 AbstractQueuedSynchronizer： parkAndCheckInterrupt() // 阻塞 当 shouldParkAfterFailedAcquire 返回true时
```
private final boolean parkAndCheckInterrupt() {
	//阻塞
    LockSupport.park(this);
    //返回线程中断状态
    return Thread.interrupted();
}
```
5.5 AbstractQueuedSynchronizer： selfInterrupt() //acquireQueued 返回true时中断当前线程

使用公平锁，调用释放锁方法 unlock() 时，方法调用轨迹：
1.ReentrantLock：unlock()
2.AbstractQueuedSynchronizer：release(int arg)
```
public final boolean release(int arg) {
	//尝试释放锁
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
        	//唤醒后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
3.Sync：tryRelease(int release)
```
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    //只有占有当前锁对象才能调用这个方法
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    //同步状态为0
    if (c == 0) {
        free = true;
        //释放锁；设置锁占有对象为null
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```
4.AbstractQueuedSynchronizer：unparkSuccessor(Node node) //Sync：tryRelease返回true时被调用
```
private void unparkSuccessor(Node node) {
    //获取节点等待状态
    //非正常状态时，设置等待状态为正常 0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    //获取下一个节点
    //如果节点为 null 或被取消
    //从队尾遍历找到一个未取消的后继节点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
    	//解除线程睡眠状态
        LockSupport.unpark(s.thread);
}
```

非公平锁的tryAcquire(int acquires) 其实调用了 Sync：nonfairTryAcquire(acquires);
```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
