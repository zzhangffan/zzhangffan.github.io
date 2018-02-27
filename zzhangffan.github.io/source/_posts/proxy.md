---
title: jdk及CGLib代理
date: 2018-02-27 14:03:00
categoreies:
tags: [CGLib,Proxy,Java]
---
> 代理：为其他对象提供一种代理以控制对这个对象的访问。


## 静态代理
程序运行前就已经存在的编译好的代理类

假设我们有一个Hello类，其中有一个sayHello的方法，输出一个字符串，代码如下：
```
public class Hello{

	public void sayHello(){
		System.out.println("hello!");
	}
}
```
现在要在sayHello方法执行前后做些事情，比如各打印一句话，我们可以将上面的程序改成这样：
```
public class Hello{

	public void sayHello(){
		System.out.println("========== begin ==========");
		System.out.println("hello!");
		System.out.println("========== end ==========");
	}
}
```
看似完成了效果，但是需要我们修改sayHello方法的代码，有没有办法不修改目标方法呢，我们可以创建一个代理类，如下：
```
public class HelloProxy{

	private Hello hello;

	public HelloProxy(Hello hello){
		this.hello = hello;
	}

	public void doProxy(){
		System.out.println("========== begin ==========");
		hello.sayHello();
		System.out.println("========== end ==========");
	}

	public static void main(String [] args) {
		HelloProxy proxy = new HelloProxy(new Hello());
		proxy.doProxy();
	}
}
```
但是，如果我们有多个类要实现同样的代理任务，按这种方式我们需要为每个类相应的创建一个XXXProxy代理类，看起来有点傻，有没有其他办法呢？我们可以用Java的动态代理来实现：

## 动态代理
程序运行前并不存在代理类，无需手动编写代理类的源码
<span class="note">Java动态代理需要代理类和委托类实现同一个接口,通过Java反射来生成。</span>
定义一个目标接口：
```
public interface TargetInterface{

	public void sayHello();
	public void sayGoodbye();
}
```
实现目标接口的委托类：
```
public class TargetImpl implements TargetInterface{

	public void sayHello(){
		System.out.println("========== hello ==========");
	}

	public void sayGoodbye(){
		System.out.println("========== goodbye ==========");
	}
}
```
最后，代理处理类：
```
public class ProxyHandler implements InvocationHandler{

	private Object target;

	public ProxyHandler(Object target){
		this.target = target;
	}

	//实现了InvocationHandler接口，方法调用会被转发到这个方法
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("========== before ==========");
		Object object = method.invoke(target, args);
		System.out.println("========== after ==========");  
		return object;
	}

	//获取代理类，*关键
	public <T> T getProxy(){
		return (T) Proxy.newProxyInstance(
			target.getClass().getClassLoader(),//代理对象的类加载器
			target.getClass().getInterfaces(),//代理对象要实现的接口
			this);//实际处理程序
	}

	public static void main(String [] args){
		TargetInterface target = new ProxyHandler(new TargetImpl()).getProxy();
		target.sayHello();
	}
}
```
反编译代理类后，如下：
```
public final class $proxy4
  extends Proxy
  implements TargetInterface
{
  private static Method m3;
  private static Method m1;
  private static Method m0;
  private static Method m4;
  private static Method m2;
  
  public $proxy4(InvocationHandler paramInvocationHandler)
    throws 
  {
    super(paramInvocationHandler);
  }
  
  public final void sayHello()
    throws 
  {
    try
    {
      this.h.invoke(this, m3, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }
 
  
  public final void sayGoodbye()
    throws 
  {
    try
    {
      this.h.invoke(this, m4, null);
      return;
    }
    catch (Error|RuntimeException localError)
    {
      throw localError;
    }
    catch (Throwable localThrowable)
    {
      throw new UndeclaredThrowableException(localThrowable);
    }
  }

  //省略来自Object类的hashCode(),equals()以及toString()方法
  
  static
  {
    try
    {
      m3 = Class.forName("com.TargetInterface").getMethod("sayHello", new Class[0]);
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      m4 = Class.forName("com.TargetInterface").getMethod("sayGoodbye", new Class[0]);
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      return;
    }
    catch (NoSuchMethodException localNoSuchMethodException)
    {
      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException)
    {
      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
}
```
前面我们说过Java原生动态代理需要代理类和委托类实现同一接口，那如果是没有接口的类呢？
我们可以使用开源的CGLib。
用我们之前定义的Hello类，他没有实现任何接口，如下：
```
public class Hello{
	public void sayHello(){
		System.out.println("hello!");
	}
}
```
代理处理类：
* 实现MethodInterceptor，方法调用转发到intercept()方法

```
public class ProxyInterceptor implements MethodInterceptor{

	@Override
	public Object intercept(Object target, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		System.out.println("========== before ==========");
		Object object = proxy.invokeSuper(target,args);
		System.out.println("========== end ==========");
		return object;
	}

	public <T> T getProxy(Class<T> cls){
		return (T) Enhancer.create(cls,this);//1 父类 2实际处理
	}

	public static void main(String[] args) {
		ProxyInterceptor proxy = new ProxyInterceptor();
		TargetInterface target = proxy.getProxy(Hello.class);
		target.sayHello();
	}
}
```
代理类具体实现（来自网络）：
```
public class HelloConcrete$$EnhancerByCGLIB$$e3734e52
  extends HelloConcrete
  implements Factory
{
  ...
  private MethodInterceptor CGLIB$CALLBACK_0; // ~~
  ...
   
  public final String sayHello(String paramString)
  {
    ...
    MethodInterceptor tmp17_14 = CGLIB$CALLBACK_0;
    if (tmp17_14 != null) {
      // 将请求转发给MethodInterceptor.intercept()方法。
      return (String)tmp17_14.intercept(this, 
              CGLIB$sayHello$0$Method, 
              new Object[] { paramString }, 
              CGLIB$sayHello$0$Proxy);
    }
    return super.sayHello(paramString);
  }
  ...
}
```

## 总结

* 使用jdk实现动态代理需要代理类和委托类实现同样的接口。
* 使用CGLib，用继承实现，不能对final修饰的类及方法使用。

（完）
======