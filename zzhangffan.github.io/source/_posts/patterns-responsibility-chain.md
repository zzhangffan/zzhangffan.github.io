---
title: 责任链模式
date: 2018-03-06 22:48:53
categoreies:
tags: 设计模式
---
> 责任链模式（行为型模式）：让多个对象都有机会处理请求，将这些对象连成一条链，处理请求时，让请求在这个链上传递，直到链式的摸一个对象处理了这个请求。避免请求的发送者与处理者耦合。

## 实现 ##
定义一个日志处理模板类：
```
public abstract class AbstractLogger {

    public static int INFO = 1;
    public static int DEBUG = 2;
    public static int ERROR = 3;

    protected int level;

    /**
     * 保存下一个处理程序引用（一个接一个形成链）
     */
    protected AbstractLogger nextLogger;

    public void setNextLogger(AbstractLogger nextLogger) {
        this.nextLogger = nextLogger;
    }

    /**
     * 模板处理方法
     */
    public void logMessage(int level, String message) {
        if (this.level <= level) {
        	//“钩子”方法
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }

    abstract protected void write(String message);
}
```
创建具体的日志处理类：
```
//文件日志
public class FileLogger extends AbstractLogger {

    public FileLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("File::Logger:" + message);
    }
}
```
```
//错误日志
public class ErrorLogger extends AbstractLogger {

    public ErrorLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Error Console::Logger:" + message);
    }
}
```
```
//控制台日志
public class ConsoleLogger extends AbstractLogger {

    public ConsoleLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Standard Console::Logger:" + message);
    }
}
```
测试程序：
```
public class ResponsibilityChainTest {

    private static AbstractLogger getChainOfLoggers(){
        AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
        AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
        AbstractLogger consoleLogger = new FileLogger(AbstractLogger.INFO);

        //串成链  errorLogger => fileLogger => consoleLogger
        errorLogger.setNextLogger(fileLogger);
        fileLogger.setNextLogger(consoleLogger);

        return errorLogger;
    }

    public static void main(String[] args) {
        //获取日志执行链
        AbstractLogger loggerChain = getChainOfLoggers();

        //分别打印不同级别的日志，
        //而不用关注具体的处理程序是哪一个，
        //只管把“打印不同级别日志”这个请求往执行链传就行
        loggerChain.logMessage(AbstractLogger.INFO,"This is an information.");

        loggerChain.logMessage(AbstractLogger.DEBUG, "This is an debug level information.");

        loggerChain.logMessage(AbstractLogger.ERROR, "This is an error information.");
    }
}
```
程序运行结果：
> File::Logger:This is an information.
File::Logger:This is an debug level information.
File::Logger:This is an debug level information.
Error Console::Logger:This is an error information.
File::Logger:This is an error information.
File::Logger:This is an error information.
>
Process finished with exit code 0

## 其他例子 ##
Servlet中Filter、FilterChain
...

（完）
=====

> 程序例子来自：[菜鸟教程](http://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)