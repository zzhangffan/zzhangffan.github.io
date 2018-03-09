---
title: Spring事务传播行为
date: 2018-03-03 15:13:08
categoreies:
tags: [Spring,Framework]
---
> Spring对事务的解决方案，是对 [数据库事务](/20180303/transaction/) 的补充或扩展。它是解决方法间的事务传播。

## 事务传播行为 ##
Spring一共提供了7种事务传播行为：
假设事务从方法A传播到了方法B，面对方法B他需要问一个问题：事务A有事务吗？
1. PROPAGATION_REQUIRED
如果没有，就新建一个事务；如果有，就加入当前事务。Spring默认事务传播行为。
2. PROPAGATION_REQUIRES_NEW
如果没有，就新建一个事务；如果有，就将当前事务挂起，创建一个新事物，与原来的事务没有任何关系。
3. PROPAGATION_NESTED
如果没有，就新建一个事务；如果有，就在当前事务中嵌套一个事务。子事务与主事务之间是有关联的，当主事务提交或回滚时，子事务也会提交或回滚。
4. PROPAGATION_SUPPORTS
如果没有，就以非事务方式执行；如果有，已使用当前事务。<note>有就有，没有就没有</note>
5. PROPAGATION_NOT_SUPPORTED
如果没有，就以非事务方式执行；如果有，就将当前事务挂起。<note>没有就算了，有也不支持</note>。
6. PROPAGATION_NEVER
如果没有，就以非事务方式执行；如果有，就抛出异常。<note>没有就算了，有也不支持，不仅不支持我还给你报错</note>
7. PROPAGATION_MANDATORY
如果没有，就抛出异常；如果有，就是用当前事务。<note>必须要有事务</note>

Spring还提供了一些其他功能：
* 事务超时
解决事务时间太久，消耗资源，给事务设置一个最大时长，超过了直接回滚事务。
* 只读事务
忽略哪些不需要事务的方法，提升性能。

## Spring事务配置 ##

### 注解式配置 ###
```
...
<tx:annotation-driven transaction-manager="事务管理器"/>
...
```
在需要事务的方法上使用：
```
@Transactional
public void xxx() {
	...
}
```
事务可配置属性：
* 事务传播性
` @Transactional(propagation=Propagation.REQUIRED) `
* 事务的超时性
` @Transactional(timeout=30) ` 单位秒
* 事务的隔离级别
` @Transactional(isolation=Isolation.READ_COMMITTED) `
* 回滚（指定需要回滚的异常类）
` @Transactional(rollbackFor={RuntimeException.class,Exception.class}) `
* 只读
` @Transactional(readOnly=true) `
...

<note>` @Transactional ` 注解定义在类上，对类中所有public方法有效，定义在方法上对被定义方法（public）有效。</note>

### XML配置 ###
```
...
<tx:advice id="TestAdvice" transaction-manager="事务管理器">  
    <!--配置事务传播性，隔离级别以及超时回滚等问题 -->  
    <tx:attributes>  
        <tx:method name="save*" propagation="REQUIRED" />  
        <tx:method name="del*" propagation="REQUIRED" />  
        <tx:method name="update*" propagation="REQUIRED" />  
        <tx:method name="add*" propagation="REQUIRED" />  
        <tx:method name="*" rollback-for="Exception" />  
    </tx:attributes>  
</tx:advice>
...
```

（完）
=====