## String a = "abc"; String b = "abc"; String c = new String("abc"); String d = "ab" + "c";他们之间用==比较的结果

答:

a == b结果为true;</br>
a == c结果为false, b == c结果为false;</br>
a == d结果为true, b == d 结果为true, c== d结果为false;


先查看如下测试代码：
```java
public class Test {
	public static void main(String[] args) {
		String a = "abc";
		String b = "abc";
		String c = new String("abc");
		String d = "ab"+"c";

		String str1 = "ab";
		String str2 = "c";
		String e = str1 + str2;

		System.out.println("a==b的结果是:"+(a == b));
		System.out.println("b==c的结果是:"+(b == c));
		System.out.println("b==d的结果是:"+(b == d));
		System.out.println("b==e的结果是:"+(b == e));
		System.out.println("c==e的结果是:"+(c == e));
	};
}
```
执行结果为
```java
a==b的结果是:true
b==c的结果是:false
b==d的结果是:true
b==e的结果是:false
c==e的结果是:false
```

分析:
- ==比较的是对象地址值
-  关于String str = "abc"的内部工作。Java内部将此语句转化为以下几个步骤：</br>
   (1)先定义一个名为str的对String类的对象引用变量：String   str;</br>
   (2)在栈中查找有没有存放值为 "abc "的地址，如果没有，则开辟一个存放字面值为 "abc "的地址，接着创建一个新的String类的对象o，并将o   的字符串值指向这个地址，而且在栈中这个地址旁边记下这个引用的对象.如果已经有了值为 "abc "的地址，则查找对象o，并返回o的地址.</br>
   (3)将str指向对象o的地址。

`String a = "abc";`在字符串池中查找不到存放"abc"的地址，所以开辟一个地址存放"abc";</br>
`String b = "abc";` 在字符串池中找到存放"abc",直接指向地址。</br>
因此a,b的地址值相同，==比较结果为true.

`String c = new String("abc");`直接新创建地址存放"abc",因此和a,b进行==比较结果为false;

`String d = "ab" + "c";`在某些特别情况下， String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接。其实这是 JVM 的一个把戏，在 JVM 眼里，这个 `String a = "ab" +  "c";` 其实就是： `String S1 = "abc";`直接引用地址值， 不需要太多的时间

但大家这里要注意的是，如果你的字符串是来自其他的 String对象的话，情况就不同了，比如下面:
```java
String str1 = "ab";
String str2 = "c";
String e = str1 + str2;
```
这时候 JVM 会规规矩矩的按照原来的方式去做，会新建地址存放结果，而且大量的无引用中间对象可能会引起GC操作，影响效率。因此a,b和e的==结果为false;

同样，c和e的==结果当然也是false，因为他们具有不同的地址值。