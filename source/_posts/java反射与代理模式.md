---
title: java反射与代理模式
date: 2018-08-25 17:29:12
tags:
- java
- proxy
- reflect
categories: 编程
---

## java.lang.Object中getClass()方法的用途
可以获取一个类的定义信息，然后使用反射去访问其全部信息(包括函数和字段)。
还可以查找该类的ClassLoader，以便检查类文件所在位置等。
例子：
```java
Class class1=xxx.getClass();
```
class1带的方法有什么用？比如说可以返回类名，知道该类中字段，知道该类中方法名，知道该类中参数名，知道该类中方法返回类型等等。

## java中getClass()和.class的联系和区别
一般情况下，getclass()方法和.class方法是等价的，都可以获得一个类型名。

区别：
两者最直接的区别就是，getClass()是一个类的实例所具备的方法，而.class方法是一个类的方法。
另外getClass()是在运行时才确定的，而.class方法是在编译时就确定了。

## 代理模式的特征与作用
特征：代理类与委托类有同样的接口

作用：代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。

## 静态代理
常规的代理模式有以下三个部分组成：

功能接口
``` java
interface IFunction {
	void doAThing();
}
```
功能提供者
``` java
class FunctionProvider implement IFunction {
	public void doAThing {
		System.out.print("do A");
	}
}
```
功能代理者
``` java
class Proxy implement IFunction {
	private IFunction function;
	Proxy(IFunction function) {
		this.function = function;
	}
	public void doAThing {
	     // 调用委托类方法前做一些操作
		function.doAThing();
		// 调用委托类方法后也可以做一些操作
	}
}
```


静态代理类：由程序员创建或由特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。

缺点：比如有多个接口需要进行代理，那么就要为每一个功能提供者创建对应的一个代理类，那就会越来越庞大。而且，所谓的“静态”代理，意味着必须提前知道被代理的委托类。


## 动态代理
动态代理类：在程序运行时，运用反射机制动态创建而成。

动态代理的核心思想是通过 Java Proxy 类，为传入进来的任意对象动态生成一个代理对象，这个代理对象默认实现了原始对象的所有接口。

```java
interface IAFunc {
	void doA();
}
interface IBFunc {
	void doB();
}
```

```java
class A implement IAFunc { ... }
class B implement IBFunc { ... }
```

```java
class TimeConsumeProxy implements InvocationHandler {
	private Object realObject;
	public Object bind(Object realObject) {
		this.realObject = realObject;
		Object proxyObject = Proxy.newInstance(
			realObject.getClass().getClassLoader(),
			realObject.getClass().getInterfaces(),
			this
		);
		return proxyObject;
	}
	@Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.currentMillions();
        Object result = method.invoke(target, args);
        System.out.println("耗时：" + (System.currentMillions() - start));
        return result;
    }
}

```
具体使用时：
```java
public static void main(String[] args) {
	A a = new A();
	IAFunc aProxy = (IAFunc) new TimeConsumeProxy().bind(a);
	aProxy.doA();
	B b = new B();
	IBFunc bProxy = (IBFunc) new TimeConsumeProxy().bind(b);
	bProxy.doB();
}
```

这里最大的区别就是：代理类和委托类互相透明独立，逻辑没有任何耦合，在运行时才绑定在一起。这也就是静态代理与动态代理最大的不同，带来的好处就是：无论委托类有多少个，代理类不受到任何影响，而且在编译时无需知道具体委托类。

## 探索动态代理实现机制
上面代码中最重要的就是:
```java
Object proxyObject = Proxy.newInstance(
			realObject.getClass().getClassLoader(),
			realObject.getClass().getInterfaces(),
			this
		);
```
通过 Proxy 工具，把真实委托类转换成了一个代理类，最开始提到了一个代理模式的三要素：功能接口、功能提供者、功能代理者；在这里对应的就是：realObject.getClass().getInterfaces()，realObject，TimeConsumeProxy。

其实动态代理并不复杂，通过一个 Proxy 工具，为委托类的接口自动生成一个代理对象，后续的函数调用都通过这个代理对象进行发起，最终会执行到 InvocationHandler#invoke 方法，在这个方法里除了调用真实委托类对应的方法，还可以做一些其他自定义的逻辑，比如上面的运行耗时统计等。


### 问题一: proxyObject 究竟是个什么 -> 动态生成的 $Proxy0.class 文件

在调用 Proxy.newInstance 后，Java 最终会为委托类 A 生成一个真实的 class 文件：$Proxy0.class，而 proxyObject 就是这个 class 的一个实例。

在IDE反编译下$Proxy0.class文件
```java
final class $Proxy0 extends Proxy implements IAFunc {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }
    public final boolean equals(Object var1) throws  {
        // 省略
    }
    public final void doA() throws  {
        try {
        	// 划重点
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    public final String toString() throws  {
        // 省略
    }
    public final int hashCode() throws  {
        // 省略
    }
    static {
        try {
        	// 划重点
            m3 = Class.forName("proxy.IAFunc").getMethod("doA", new Class[0]);
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```
其中doA()里做的就是调用 TimeConsumeProxy#invoke() 方法。

那么也就是说，下面这段代码执行流程如下：

```java
IAFunc aProxy = (IAFunc) new TimeConsumeProxy().bind(a);
aProxy.doA();
```
基于传入的委托类 A，生成一个$Proxy0.class 文件；
创建一个 $Proxy0.class 对象，转型为 IAFunc 接口；
调用 aProxy.doA() 时，自动调用 TimeConsumeProxy 内部的 invoke 方法。

### 问题二：proxyObject 是怎么一步步生成出来的 -> $Proxy0.class 文件生成流程

刚刚从末尾看了结果，现在我们回到代码的起始端来看：

```java
Object proxyObject = Proxy.newInstance(
			realObject.getClass().getClassLoader(),
			realObject.getClass().getInterfaces(),
			this
		);
```

先看Proxy.newInstance():

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h) {
    // 复制要代理的接口
	final Class<?>[] intfs = interfaces.clone();
	// 重点：生成 $Proxy0.class 文件并通过 ClassLoader 加载进来
	Class<?> cl = getProxyClass0(loader, intfs);
	// 对 $Proxy0.class 生成一个实例，就是 `proxyObject`
	final Constructor<?> cons = cl.getConstructor(constructorParams);
	return cons.newInstance(new Object[]{h});
}
```

再来看 getProxyClass0 的具体实现：ProxyClassFactory工厂类：

```java
@Override
public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
	// 参数为 ClassLoader 和要代理的接口
	Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
	// 1. 验证 ClassLoader 和接口有效性
	for (Class<?> intf : interfaces) {
		// 验证 classLoader 正确性
		Class<?> interfaceClass = Class.forName(intf.getName(), false, loader);
		if (interfaceClass != intf) {
            throw new IllegalArgumentException(
                intf + " is not visible from class loader");
        }
		// 验证传入的接口 class 有效
		if (!interfaceClass.isInterface()) { ... }
		// 验证接口是否重复
		if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) { ... }
	}
	// 2. 创建包名及类名 $Proxy0.class
	proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
	long num = nextUniqueNumber.getAndIncrement();
    String proxyName = proxyPkg + proxyClassNamePrefix + num;
    // 3. 创建 class 字节码内容
    byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
    // 4. 基于字节码和类名，生成 Class<?> 对象
    return defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length);
}
```
再看下第三步生成 class 内容 ProxyGenerator.generateProxyClass：
```java
// 添加 hashCode equals toString 方法
addProxyMethod(hashCodeMethod, Object.class);
addProxyMethod(equalsMethod, Object.class);
addProxyMethod(toStringMethod, Object.class);
// 添加委托类的接口实现
for (int i = 0; i < interfaces.length; i++) {
    Method[] methods = interfaces[i].getMethods();
    for (int j = 0; j < methods.length; j++) {
         addProxyMethod(methods[j], interfaces[i]);
    }
}
// 添加构造函数
methods.add(this.generateConstructor());
```
这里构造好了类的内容：添加必要的函数，实现接口，构造函数等，下面就是要写入上一步看到的 $Proxy0.class 了。
```java
ByteArrayOutputStream bout = new ByteArrayOutputStream();
DataOutputStream dout = new DataOutputStream(bout);
dout.writeInt(0xCAFEBABE);
...
dout.writeShort(ACC_PUBLIC | ACC_FINAL | ACC_SUPER);
...
return bout.toByteArray();
```
### 动态代理小结

通过上面的讲解可以看出，动态代理可以随时为任意的委托类进行代理，并可以在 InvocationHandler#invoke 拿到运行时的信息，并可以做一些切面处理。

在动态代理背后，其实是为一个委托类动态生成了一个 $Proxy0.class 的代理类，该代理类会实现委托类的接口，并把接口调用转发到 InvocationHandler#invoke 上，最终调用到真实委托类的对应方法。

动态代理机制把委托类和代理类进行了隔离，提高了扩展性。


参考: [文章](http://wingjay.com/2018/02/11/java-dynamic-proxy/)
