## 什么是反射
反射是在运行状态当中，对于任意一个类，都可以知道这个类的所有属性和方法，对于任意一个对象，都可以调用它的任意一个方法和属性，这样的动态获取的信息以及动态调用对象的方法的功能被叫做java语言的反射机制。

作用：
- 在运行的时候构造任意一个类的对象;
- 在运行的时候判断任意一个对象所属的类;
- 在运行的时候任意调用一个对象的方法;
- 在运行的时候判断任意一个类所具有的成员变量以及方法。

## 反射中Class.forName和classloader的区别

Class.forName默认加载并初始化class，即会执行静态代码块和初始化静态变量。</br>
ClassLoader默认只加载类进JVM，并不初始化。

例如JDBC链接，就需要使用Class.forName，因为需要执行静态代码块注册驱动到DriverManager上，之后才能通过DriverManager获取连接。

Class.forName("com.mysql.jdbc.Driver");//通过这种方式将驱动注册到驱动管理器上。

这里要说明一下`静态代码块的执行问题`，以及`字节码对象的获取方式`

## 静态代码块的执行问题

注意，类加载的时候可以执行静态代码块，但是不一定执行静态代码块。

静态代码块的执行，严格来讲，是在类加载的时候，在执行初始化时，才会执行静态代码块，也就是在初始化这个类的时候才执行静态代码块。

## 字节码对象的获取方式?(前三种是常用方式)

a)类名.class，`不会加执行态代码块`
```java
Class<Object> c1 =Object.class
```
b)Class.forName(“包名.类名”) ,`会执行静态代码块`
```java
Class<?> c2 = Class.forName("java.lang.Object");
```
c)类的实例对象.getClass(),`会执行静态代码块`
```java
Class<?> c3 = new Object().getClass();
```
d)Class.forName("包名.类名", boolean,loader)
```java
Class<?> c4 = Class.forName("com.java.oop.ClassA", false, ClassLoader.getSystemClassLoader());
		//boolean值中 true为执行初始化，false为不执行初始化。
		//如果为false的话不会执行静态代码块，为true会执行
		//ClassLoader.getSystemClassLoader()这里需要传入一个类加载器，我们调用的系统默认类加载器
```
e)类加载器.load("包名+类名") `不会执行静态代码块`
```java
ClassLoader loader = ClassLoader.getSystemClassLoader();
		Class<?> c5 = loader.loadClass("com.java.oop.ClassA");//不会执行静态代码块。
```

