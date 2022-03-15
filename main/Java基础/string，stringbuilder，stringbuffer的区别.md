# string，stringbuilder，stringbuffer的区别
- `String`：字符串常量（不可变）
- `StringBuilder`：字符串变量（非线程安全）
- `StringBuffer`： 字符串变量（线程安全）
- 大部分情况下的效率对比是: StringBuilder > StringBuffer > String

**String**</br>
String 是`不可变的对象`, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个`新的`String 对象，然后将指针指向新的 String 对象，原始的String对象并没有改变。

所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。

而如果是使用 StringBuffer类,则结果就不一样了，每次结果都会对 StringBuffer`对象本身`进行操作，而不是生成新的对象再改变对象引用。

>但是在某些特别情况下， String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢。比如以下的字符串对象生成中，String 效率是远要比 StringBuffer 快的：
>```java
>   String S1 = “This is only a” + “ simple” + “ test”;
>   ```
>```java
>StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
>```
>你会很惊讶的发现，生成 String S1 对象的速度简直太快了，而这个时候 StringBuffer 居然速度上根本一点都不占优势<br/>
> 其实这是 JVM 的一个把戏，在 JVM 眼里，这个<br/>
`String S1 = “This is only a” + “ simple” + “test”;` 其实就是：
`String S1 = “This is only a simple test”;` <br/>所以当然不需要太多的时间了。但大家这里要注意的是，如果你的字符串是来自另外的 String 对象的话，速度就没那么快了，譬如：
>```java
>String S2 = “This is only a”;
>String S3 = “ simple”;
>String S4 = “ test”;
>String S1 = S2 +S3 + S4;
>```
>这时候 JVM 会规规矩矩的按照原来的方式去做。大量的无引用中间对象可能会引起GC操作，影响效率。

**StringBuffer**</br>
`Java.lang.StringBuffer`线程安全的可变字符序列。一个类似于 String 的字符串缓冲区，但不能修改。虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。

StringBuffer 上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。</br>
append 方法始终将这些字符添加到缓冲区的末端；</br>
insert 方法则在指定的点添加字符。
>例如，如果 z 引用一个当前内容是“start”的字符串缓冲区对象，则此方法调用 z.append(“le”) 会使字符串缓冲区包含“startle”，而 z.insert(4, “le”) 将更改字符串缓冲区，使之包含“starlet”。

**StringBuilder**</br>
`java.lang.StringBuilder`一个可变的字符序列,他是是JDK1.5新增的。</br>
此类提供一个与 StringBuffer 兼容的 API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。</br>
如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同。