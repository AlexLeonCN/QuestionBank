## String类为什么是final的？

**答：**
- 1.为了实现字符串池
- 2.为了线程安全
- 3.为了实现String可以创建HashCode不可变性
- 4.为了系统安全

**解析：**

首先，final关键字可以修饰类，方法和变量：
- 被final修饰的类不能被继承，即它不能拥有自己的子类;
- 被final修饰的方法不能被重写;
- final修饰的变量，无论是类变量、实例变量、参数变量(形参)还是局部变量，都需要进行初始化操作。

在了解final的用途后，再看String为什么要被final修饰：主要是为了"`安全性`"和"`效率`"的缘故。

查看JDK String的源码:

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    //...
    }
```

>Java源码的注释是：
>
>String类表示字符串。Java程序中的所有字符串文本，例如“abc”，都是作为此类的实例实现的。字符串是常量；它们的值在创建后不能更改。String buffers缓冲区支持可变字符串。因为字符串对象是不可变的，所以可以共享它们。

final修饰的String，代表了String的不可继承性，这就避免了因为继承引起的安全隐患；

final修饰的char[]代表了被存储的数据不可更改性；

但是，虽然final修饰代表了不可变，但仅仅是==引用地址==不可变，并不代表了数组本身不会变,请看下面代码。

```java
public class Test {
	public static void main(String[] args) {
		 final int[] arr = {1,2,3,4,5};
		 System.out.println(arr);//地址值 [I@15db9742
		 arr[3] = 100;
		 for (int i : arr) {
			System.out.println(i);//{1,2,3,100,5}
		}
		 System.out.println(arr);//地址值 [I@15db9742
	}
}
```

所以，即便使用final修饰，数组本身也可以改变的，只是引用地址值不变。

同时起作用的还有private，正是因为两者保证了String的不可变性。

- 那为什么要保证String不可变呢？

只有字符串是不可变的，`字符串池`才有可能实现。`不同的字符串变量可以指向池中的同一个字符串`，节省heap空间。但如果字符串是可变的，如果变量改变了它的值，那么其它指向这个值的变量的值也会一起改变。

只有字符串是不可变的，`多线程才安全`，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步，字符串本身就是线程安全的。

只有字符串是不可变的，则在它`创建的时候HashCode就可以被允许缓存`，并且不会在每次调用 String 的 hashcode 方法时重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

而且String类中的很多方法的实现不是Java代码，而是调用`操作系统的本地方法来`完成的，如果String类不被final修饰，被继承重写方法的话，系统会很不安全。

同时，String 不可变的绝对最重要的原因是它被`类加载机制`使用，因此具有深刻和基本的安全考虑。

设计成final修饰，JVM才不用对相关方法在虚函数表中查询，而直接定位到String类的相关方法上，提高了执行效率。