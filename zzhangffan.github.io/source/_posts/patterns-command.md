---
title: 命令模式
date: 2018-03-06 23:39:13
categoreies:
tags: 设计模式
---
> 命令模式（行为型模式）：封装命令，将命令的发出者与执行者之间解耦。

## 实现 ##
创建命令执行者类：
```
public class Receiver {

    public void action() {
        System.out.println("Receiver::action");
    }
}
```
创建命令接口：
```
public interface Command {

    void execute();
}
```
创建实际命令类：
```
public class ConcreteCommand implements Command {

    private Receiver receiver;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.action();
    }
}
```
创建命令发出者类：
```
public class Invoker {

    private Command command;

    public Invoker(Command command) {
        this.command = command;
    }

    public void action() {
        command.execute();
    }
}
```
测试：
```
public class CommandTest {

    public static void main(String[] args) {
        Receiver receiver = new Receiver();

        //通过命令将发出者和执行者解耦
        Command cmd = new ConcreteCommand(receiver);

        Invoker invoker = new Invoker(cmd);
        invoker.action();
    }
}
```
程序运行结果：
> Receiver::action
>
Process finished with exit code 0

（完）
=====