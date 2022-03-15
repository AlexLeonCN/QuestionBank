# Java7、Java8中的新特性
## Java 7的新特性

- switch中可以使用字串了
- 运用List tempList = new ArrayList<>(); 即泛型实例化类型自动推断

## Java 8的新特性

- 允许接口中包含具有默认实现的方法，这种方法被称为“默认方法”，使用default关键字修饰
- Lambda表达式，需要函数式接口的支持。</br>
  Lambda允许把函数作为一个方法的参数(函数作为参数传递进方法中)</br>
  Lambda表达式可以分为两部分：</br>
  左侧：指定了Lambda表达式需要的所有参数</br>
  右侧：指定了Lambda体，即lambda表达式要执行的功能
- 函数式接口<br>
  所谓的函数式接口就是必须有且仅有一个抽象方法声明的接口，Lambda表达式必须要与抽象方法的声明相匹配，由于默认方法不是抽象的，所以可以在函数式接口中任意添加默认方法。
- Stream 数据流API：数据渠道，用于操作数据源（集合，数组等）所生成的元素序列“集合讲的是数据，流讲的是计算”。</br>
  新添加的Stream API（java.util.stream）把真正的函数式编程风格引入到Java中</br>
  注意：</br>
  Stream自己不会存储元素</br>
  Stream不会改变源对象。相反，他们会返回一个持有结果的新的Stream</br>
  Stream操作是延迟执行的。这意味着他们会等到需要结果的时候才会执行。
- Optional类</br>
  Optional类是一个容器类，代表一个值存在或不存在，可以避免空指针异常。</br>
  Optional不是一个函数式接口，而是一个精巧的工具接口，用来防止NullPointException异常。</br>
  Optional是一个简单的值容器，这个值可以是null，也可以是non-null，为了不直接返回null，我们在Java 8中就返回一个Optional类，使用Optional类可以避免使用if进行null检查，这种检查的过程交由底层自动处理。主要有如下一些方法：
    ```java
    of(T t) : 创建一个Optional实例
    empty() : 创建一个空的Optional实例
    ofNullable(T t) :  若t不为null，创建Optional实例，否则创建空的Optional实例
    isPresent() : 判断是否包含值
    orElse(T t) : 如果调用对象包含值，返回该值，否则返回t
    orElseGet(Suppplier s) : 如果调用对象包含值，返回该值，否则返回s获取的值
    map(Function f) : 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty()
    flatMap(Function mapper) : 与map类似，要求返回值必须是Optional
    ```
- JAVA8全新的时间包</br>
  Java 8 包含了全新的时间日期API，这些功能都放在了java.time包下。主要有如下一些方法：
    ```java
    LocalDate，LocalTime，LocalDateTime ： 时间
    Instant ： 时间戳（以Unix元年：1970年1月一日 00:00:00到某个时间时间的毫秒值）
    Duration ： 计算两个“时间”之间的间隔
    Period ： 计算两个“日期”之间的间隔
    TemporalAdjust ：时间校正器
    DateTimeFormatter ：格式化时间/日期
    ZonedDate，ZonedTime，ZonedDateTime ： 时区
    ```
- HashMap的bucket的链表中Node节点数增加到8(默认阈值)会转化为红黑树结构，红黑树中Node节点数如果减少到6会转化为链表结构。 降低时间复杂度，提高效率，同时避免多线程环境下形成环形链表造成死链。