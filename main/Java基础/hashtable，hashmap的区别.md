# hashtable，hashmap的区别

HashMap 与 Hashtable 的区别类似于 ArrayList 与 Vector 的区别。</br>
Hashtable 与 Vector 都是 JDK 1.0 就有一个一个古老的集合，因此 Hashtable 是一个继承自 Dictionary 的古老集合。</br>
从 JDK 1.2 引入集合框架的 Map 接口之后，Java 让 Hashtable 也实现了 Map 接口，因此 Hashtable 也新增实现了一些 Map 接口中定义的方法。</br>
实际上 Hashtable 与 HashMap底层的实现很相似，它们都是基于 Hash 表的实现。

HashMap 与 Hashtable 的区别主要有如下两点：
- A．HashMap 允许使用 null 作为 key 或 value，而 Hashtable 不允许。
- B．HashMap 是线程不安全的，性能较好；但 Hashtable 是线程安全的，性能较差。

实际上，即使在多线程环境下，Java 提供了 Collections 工具类把 HashMap 包装成线程安全的类，因此依然应该使用 HashMap，如下代码所示：
```java
Map map = Collections. synchronizedMap(new HashMap());
```
简单的说，编程时应该尽量避免使用 Hashtable，除非在一个古老的 API 中强制要求Hashtable。