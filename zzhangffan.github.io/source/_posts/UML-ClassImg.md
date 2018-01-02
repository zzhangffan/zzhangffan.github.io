---
title: UML类图
date: 2018-01-02 16:12:58
categoreies: 其他
tags: UML
---
最近在看设计模式的过程中发现用UML类图的地方特别多，学习了之后现在把它记录下来

## 先上一个最基本的类图示例
![class](/images/UML-ClassImg/class.png)
> ## 类的表示形式

在UML类图中，类由一个矩形图案表示，从上到下分别为类名、属性以及方法。

1. 类名
<span style="color: red;font-size: 20px;">
普通类用 __ ` 正体字 ` __ 书写 | 接口名称上方加 ` << interface >> ` 表示 | 抽象类用 __ ` 斜体字 ` __ 书写</span>

2. 属性
属性的表示方法如下：
<font color="blue">&emsp;&emsp;访问级别&emsp;&emsp;属性名&emsp;：&emsp;数据类型&emsp;\[ = 缺省值\]</font>
其中访问级别： 
&emsp;&emsp;— ：私有 private
&emsp;&emsp;\+ ：公共 public
&emsp;&emsp;\# ：受保护 protected
\[ = 缺省值\] 为可选项

3. 方法
类的方法的表示方法如下：
<font color="blue">&emsp;&emsp;访问级别&emsp;&emsp;名称(参数列表)&emsp;\[ ： 返回类型\]</font>

> ## 类与类之间的关系

类与类之间关系大致可分为：
- 依赖
- 关联
- 聚合
- 组合
- 继承

1. 依赖
代码表现为：类A的方法中以局部变量、方法参数的形式存在类B的对象或调用类B的静态方法，我们说类A依赖于类B，在UML类图中用虚线箭头表示
![依赖示意图.png](/images/UML-ClassImg/yilai.png)

2. 关联
代码表现为：类A中以成员变量的形式存在类B的对象，get/set赋值，我们说类A与类B是关联的状态，<span style="color:red;">关联关系强调的是平等关系</span>，在UML类图中用实线箭头表示
![关联示意图.png](/images/UML-ClassImg/guanlian.png)

3. 聚合
聚合的代码表现形式与关联一致，构造函数赋值，不好区分，区别在于包含关系的强弱，<span style="color:red;">聚合关系强调的是一方包含另一方，另一方可以单独出去，如汽车和轮胎</span>，在UML类图中用空心菱形箭头表示
![聚合示意图.png](/images/UML-ClassImg/juhe.png)

4. 组合
组合的代码表现形式与关联和聚合一致，<span style="color:red;">组合关系强调的是一方和另一方是捆绑的无法单独出去，如人和大脑</span>，在UML类图中用实心菱形箭头表示
![组合示意图.png](/images/UML-ClassImg/zuhe.png)

5. 继承
* 泛化
代码中以 ` extends ` 关键字体现，用实线三家箭头表示
![泛化示意图.png](/images/UML-ClassImg/fanhua.png)

* 实现
代码中以 ` inplements ` 关键字体现，用虚线三角箭头表示
![实现示意图.png](/images/UML-ClassImg/shixian.png)