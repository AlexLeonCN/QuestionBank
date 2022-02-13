# HashTable，HashMap和ConcurrentHashMap 底层实现原理和线程安全问题

前面已经详细分析过[HashMap的源码及实现与原理](https://github.com/AlexLeonCN/QuestionBank/blob/main/Java基础/HashMap的源码，实现原理和底层实现是怎样的%3F.md)

**HashTable**是用同步关键字Synchronized保证`线程安全`</br>
在线程竞争激烈的情况下，效率非常低。</br>
>例如如线程1使用put方法添加元素，线程2不但不能使用put添加元素，也不能使用get获取元素。

**HashMap**是`非线程安全的`。在多线程环境下使用put操作会引起死循环，导致CPU利用率接近100%，因为多线程会导致HashMap的Entry链表形成环形数据结构，Entry的next节点永远不为空，就会产生死循环获取Entry。

**JDK1.7中ConcurrentHashMap实现**：</br>
使用`分段锁技术控制并发访问`</br>
首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。因此当多个线程访问不同数据段的数据时，线程间不存在锁竞争，从而有效提高并发访问效率。

**JDK1.7里ConcurrentHashMap数据结构**：</br>
之前了解过HashMap是以链表数组存储的，数组下标表示通过Key的hash值再取余计算得到数组下标位置，key和value的Node节点就存在该数组位置下标bucket的链表里，每个节点包含key，value，和next节点。

ConcurrentHashMap是由Segment数组和bucket数组构成（bucket数组可以理解成一个HashMap）。<br/>
Segment是一种`可重入锁`（ReetrantLock），在ConcurrentHashMap里扮演锁的角色。</br>
bucket中的链表里的Entry(Node)用于存键值对数据。<br/>
一个ConcurrentHashMap包含一个Segment数组，segment的结构和HashMap类似，是链表数组结构。<br/>
一个segment包含一个bucket数组，每个bucket是一个链表结构的元素。每个Segment守护着一个bucket数组里的元素<br/>
当对数据进行修改时，必须首先获得与它对应的Segment锁。

>如图：
>![mskt_11](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_11.png)