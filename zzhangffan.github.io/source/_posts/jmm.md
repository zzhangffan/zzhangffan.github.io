---
title: Java内存模型
date: 2018-03-08 08:22:12
categoreies:
tags: Java
---
> Java内存模型定义了JVM操作内存的工作方式。

## 指令重排序 ##
<note>x86处理器只会对写-读操作做指令重排序！</note>

### 数据依赖性 ###
> 假设两个操作访问同一个变量，并且其中一个操作为写操作，那么这两个操作存在数据依赖性。

写后读	a = 1;b = a;	写一个变量之后，再读这个位置。
写后写	a = 1;a = 2;	写一个变量之后，再写这个变量。
读后写	a = b;b = 1;	读一个变量之后，再写这个变量。

### as-if-serial语义 ###
> as-if-serial语义指：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守as-if-serial语义。
>
为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果

### happens-before程序顺序规则 ###
类似 ` A happens-before B ` ，JMM 要求前一个操作（执行的结果）对后一个操作可见，且前一个操作顺序在后一个操作之前。

happens-before具有规则传递：
再有 ` B happens-before C ` JMM 要求保证 ` A happens-before C ` 。


## 内存屏障 ##
硬件内存屏障分为 读屏障、写屏障两种。

### 作用 ###
* 阻止屏障两侧的指令重排序

Java内存屏障分为：
1. LoadLoad
类似语句 ` Load1;LoadLoad;Load2 ` 在Load2或后续操作读取前，保证Load1已读取完成。
2. LoadStore
类似语句 ` Load1;LoadStore;Store2 ` 在Store2或后续写入操作前，保证Load1已读取完成。
3. StoreStore
类似语句 ` Store1;StoreStore;Store2 ` 在Store2或后续写入操作前，保证Store1的写入对其他处理器可见。
4. StoreLoad
类似语句 ` Store1;StoreLoad;Load2 ` 在Load2或后续读取操作前，保证Store1的写入对其他处理器可见。

## volatile ##

### volatile操作的内存语义 ###
> 当写一个volatile变量时，JMM 会把该线程对应的本地内存中的共享变量值刷新到主内存。当读一个volatile变量时，JMM 会将线程对应的本地内存置为无效，接下来线程将从主内存去读取共享变量。

## 锁 ##

### 锁释放和获取的内存语义 ###
> 当线程释放锁时，JMM 会把该线程对应的本地内存中的共享变量刷新到主内存。当线程获取锁时，JMM 会将线程对应的本地内存置为无效。从而使被监视器保护的临界区代码必须从主内存去读取共享变量。