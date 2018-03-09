---
title: 观察者模式
date: 2018-03-06 22:08:01
categoreies:
tags: 设计模式
---
> 观察者模式（行为型模式）：观察者观察被观察目标类的状态，被观察目标类管理所有观察它的组件，当自身状态发生改变时，通知所有的观察者组件。

## 实现 ##
定义被观察目标：
```
public class Subject {

    private List<Observer> observers = new ArrayList<Observer>();

    private int state;

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObservers();
    }

    /**
     *	添加观察者组件
     */
    public void attach(Observer observer){
        observers.add(observer);
    }

    /**
     * 通知所有观察者
     */
    public void notifyAllObservers(){
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```
定义一个观察者模板，定义一个统一的提供给观察目标实现通知的模板方法：
```
public abstract class Observer {

    protected Subject subject;
    public abstract void update();
}
```
创建多个观察者组件：
```
public class BinaryObserver extends Observer {

    public BinaryObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("Binary String:" + Integer.toBinaryString(subject.getState()));
    }
}
```
```
public class OctalObserver extends Observer {

    public OctalObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("Octal String:" + Integer.toOctalString(subject.getState()));
    }
}
```
```
public class HexObserver extends Observer {

    public HexObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("Hex String:" + Integer.toHexString(subject.getState()).toUpperCase());
    }
}
```
测试类：
```
public class ObserverTest {

    public static void main(String[] args) {
        //观察的目标对象
        Subject subject = new Subject();

        //创建观察者实体
        new HexObserver(subject);
        new OctalObserver(subject);
        new BinaryObserver(subject);

        //验证目标对象状态更改，将通知所有观察者
        System.out.println("First state change: 15");
        subject.setState(15);
        System.out.println("Second state change: 10");
        subject.setState(10);
    }

}
```
程序输出：
> First state change: 15
Hex String:F
Octal String:17
Binary String:1111
Second state change: 10
Hex String:A
Octal String:12
Binary String:1010
>
Process finished with exit code 0


（完）
=====

> 程序例子来自：[菜鸟教程](http://www.runoob.com/design-pattern/observer-pattern.html)