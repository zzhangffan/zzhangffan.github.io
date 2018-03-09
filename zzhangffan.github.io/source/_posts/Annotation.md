---
title: 自定义注解
date: 2018-03-09 15:36:02
categoreies:
tags: Java
password: 123456
---

## 自定义注解 ##
看一个示例：
```
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface SelfAnnotation {
    String value() default "";
}
```
* 使用 ` @interface ` 来声明一个注解。
* 注解类中的“方法”（成员变量）以无参无异常的方式声明，可以使用 default 指定默认值。
* 注解中的成员类型是有限制的，合法的有基本数据类型和 String , Class , Annotation , Enumeration等。
* 注解类可以没有成员，没有成员的注解成为标识注解（就跟标识接口差不多）。

__元注解:__
注解类上的注解称为元注解：
```
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
```

第一行： @Target 指明这个注解的作用域，` ElementType ` 是一个包含注解作用域的枚举类，其中类型包括：
` TYPE（类、接口） FIELD（字段声明）METHOD（方法声明）PARAMETER（参数声明）CONSTRUCTOR（构造函数声明）LOCAL_VARIABLE（局部变量声明）ANNOTATION_TYPE（注解声明）PACKAGE（包声明） `

第二行： @Retention 指定注解的生命周期，这里 ` RetentionPolicy ` 同样是一个枚举类，它囊括了注解的生命周期，其中包括：
` SOURCE（源码显示，编译时丢弃）CLASS（编译保留，运行时丢弃）RUNTIME（运行时存在，可反射获取）`

第三行： @Inherited 注解表示此注解可以被继承（一个类被此注解修饰的注解修饰时，其子类自动继承该注解）

第四行： @Documented 表示生成 javadoc 时包含注解信息

## 使用注解 ##
```
@SelfAnnotation("我在类上")
public class SelfAnnotationTest {

    @SelfAnnotation("我在方法上")
    public void hasAnnotationMethod(){
        System.out.println("我是有注解的方法");
    }

    public void otherMethod(){
        System.out.println("我没有注解");
    }
}
```
上面代码使用了刚才自定义的注解修饰了类和方法，其他的例如属性则无法被修饰。

## 解析注解 ##
```java
public class AnnotationProcessor {

    public static void main(String[] args) {
        //加载指定的类
        try {
            Class cla = Class.forName("test.SelfAnnotationTest");
            //被指定注解修饰
            if (cla.isAnnotationPresent(SelfAnnotation.class)) {
                //获取类上注解信息
                SelfAnnotation selfAnnotation
                        = (SelfAnnotation) cla.getAnnotation(SelfAnnotation.class);
                System.out.println(selfAnnotation.value());

                //解析方法上注解信息
                //获取所有方法
                Method[] methods = cla.getDeclaredMethods();
                for (Method method : methods) {
                    if (method.isAnnotationPresent(SelfAnnotation.class)) {
                        //执行被指定注解修饰的方法
                        method.invoke(new SelfAnnotationTest(), null);
                    }
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

这里的处理有些简单了，其实我们可以做很多事情。

程序运行结果：
> 我在类上
我是有注解的方法

（完）
=====