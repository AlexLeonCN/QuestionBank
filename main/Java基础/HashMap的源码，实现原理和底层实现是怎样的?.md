## HashMap的源码，实现原理和底层实现是怎样的？

**答**：

Hashmap底层是以链表数组实现的(数组+单向链表)

数组并不保存键本身，也不直接保存值，而是保存key和value节点Node的链表<br/>
Node是一个单向链表的一个结点，是HashMap的一个静态的内部类,实现了Map.Entry<K,V>接口<br/>

针对put()方法的源码，我们可以看出Hash表通过key的hashcode()得到的hash值，然后 hashCode%n取余，就是数组中所在的位置索引<br/>
n为数组容量，这种方法用以保证计算出来的值都在数组size范围内。也就是key和Value存储在key的散列码对应的那个数组下标的链表里面<br/>

由于不同的key有可能产生相同的散列码，这个冲突由链表解决，如果key的equals()也相同，那直接替换oldVaue；如果不相同，那将value add到链表的最后

取值也是一样，先查询key的散列码对应的链表是否存在，存在就循环链表里每一个元素，取出与key相同(equals())的那个元素。

在使用HashMap进行数据保存的时候，如果链表中保存的数据个数没有超过阈值8`(TREEIFY_THRESHOLD)`，那么会按照链表的形式进行存储，如果超过了这个阈值，则会将链表转为红黑树以实现树的平衡，并且利用左旋与右旋保证数据的查询性能<br/>
这个是JDK1.8的新特性，降低时间复杂度，提高效率，同时避免多线程环境下变为环形链表造成死链。

当红黑树中的Node数量减少到6时，会转化为链表结构

在HashMap类里面提供有一个`"DEFAULT_INITIAL_CAPACITY"`常量，作为初始化的容量配置，而后这个常量的默认大小为16，也就是说，数组默认可以保存的最大内容是16。

当保存内容的容量超过阈值`(DEFAULT_LOAD_FACTOR = 0.75f)`，相当于"容量*阈值" = 12，即保存12个元素的时候就会进行容量的扩充；<br/>
在进行扩充的时候HashMap采用的是成倍的扩充模式，即：每次都扩充2倍的容量(putVal()方法中reSize()方法中规定了<<1扩容时左移一位)

<hr/>

- **HashMap中put方法的过程**
1. 对key求hash值，然后再计算下标
2. 如果没有碰撞，直接存入bucket中
3. 如果碰撞了， 以链表的形式添加在后面
4. 如果链表长度超过阈值(TREEIFY_THRESHOLD==8),就把链表转成红黑树。
5. 如果节点已经存在就替换成新值并返回旧值。
6. 如果数组满了(初始容量*负载因子)，需要resize()方法进行双倍扩容并重组。

![mskt_02](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_02.png)

<hr/>

**详细解析**

- **1.hash算法的介绍**

HashMap是Map接口之中最为常见的一个类，该类的主要特点是`无序存储`，通过Java文档首先来观察一下HashMap子类的定义形式：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```
该类的定义继承形式符合之前的集合定义形式，依然提供有抽象类并且依然需要重复实现Map接口

![mskt_01](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_01.png)

通过HashMap实例化的Map接口可以针对key或value保存`null`的数据，hashMap支持null key，null value，key==null的时候，其hash值为0

保存数据的key`重复`也不会出现错误，而是出现内容的`替换`

Map它是`不能保证顺序的`，特别是，它不能保证随着时间推移保持顺序不变。

在设置了`相同的key`的内容时，put()方法会返回`旧的`数据内容。

HashMap是一`不安全`的集合，即当多线程访问的时候，同一时刻如果无法保证只有一个线程修改HashMap,则会毁坏HashMap，则抛出`ConcurrentModificationException`

<hr/>

- **2.HashMap源码分析**

**了解内部属性**

```java
public class HashMap<K,V> extends AbstractMap<K,V>
implements Map<K,V>, Cloneable, Serializable {
// 支持可序列化
private static final long serialVersionUID = 362498820763181265L;
//构建的时候默认的初始化容量=16，使用位移，2进制。
//因为2进制效率更高，1左移4位，即2^4=16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
//这个是最大容量,1左移30位，即2^30
static final int MAXIMUM_CAPACITY = 1 << 30;
//这个就是默认的负载因子，默认0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//当链表上的结点数大于这个阈值时会转成红黑树
static final int TREEIFY_THRESHOLD = 8;
//当链表上的节点数减小于这个值时树转链表
static final int UNTREEIFY_THRESHOLD = 6;
//链表结构转化为红黑树的时候，table最小容量
static final int MIN_TREEIFY_CAPACITY = 64;
```

HashMap有2个参数影响它的性能：`初始容量`和`负载因子`(initial capacity and load factor)

(1) `initial capacity`：HashMap创建的时候初始化容量，默认16，即数组默认可以保存的最大内容是16;

(2) `load factor`：当保存的内容的容量超过阈值(`DEFAULT_LOAD_FACTOR = 0.75f`)，相当于"容量*阈值" = 12，即保存12个元素的时候就会进行2倍容量的扩充重组;

<hr/>

**构造方法**
```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

//空参构造方法
 public HashMap() {
    //空参默认时，负载因子0.75f的赋值
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//int类型参数构造方法
public HashMap(int initialCapacity) {
    //加载因子还是默认赋值0.75
    //可以规定初始化容量
    //然后将参数带入，调用两个参数的构造函数
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//两个参数(负载因子和容量)构造方法
public HashMap(int initialCapacity, float loadFactor) {
    //判断容量是否合规(要求大于0)
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //判断容量是否大于最大容量(1<<30)
    //如果初始化的容量大于最大的容量，则设为最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //判断负载因子是否大于0
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    //调整新数组的阈值(capacity * loadFactory)
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
    
static final int tableSizeFor(int cap) {
    /*
    这里解析下tableSizeFor(int cap):
    >>>是无符号右移操作，|是位或操作，
    经过五次右移和位或操作可以保证得到大小为2^k-1的数
    0 0 0 0 1 ? ? ? ? ? //n
    0 0 0 0 1 1 ? ? ? ? //n |= n >>> 1;
    0 0 0 0 1 1 1 1 ? ? //n |= n >>> 2;
    0 0 0 0 1 1 1 1 1 1 //n |= n >>> 4;
    */

    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
} 
```

`HashMap只有在第一次使用put()方法时才进行空间分配(2^n)`

<hr/>

- HashMap的内部结构
  
在HashMap之中进行数据存储的依然是利用了`Node`类完成的，那么这种情况下证明可以使用的数据结构两种: `链表`(时间复杂度"O(n)")、`二叉树`(时间复杂度"O(logn)");

- **transient Node<K,V>[] table;**
  
![mskt_03](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_03.png)

Node是一个单向链表的一个结点，是HashMap的一个静态的内部类,实现了`Map.Entry<K,V>`接口。

包括了健，值，下个节点的引用。

源码如下：

```java
static  class  Node<K,V> implements  Map.Entry<K,V> {
    final  int  hash;
    final  K key;  //当前节点的key值
    V value;  //当前节点的value值
    Node<K,V> next;  //下一个Node节点的引用
    Node(int  hash, K key, V value, Node<K,V> next) {
        this.hash  = hash;
        this.key  = key;
        this.value  = value;
        this.next  = next;
    }
    public  final  K getKey()     { return  key; }
    public  final  V getValue()   { return  value; }
    public  final  String toString() { return  key  + "="  + value; }
    //这个Node节点的hashCode值，是key和value分别的hashCode值的异或关系。
    public  final  int  hashCode() {
        return  Objects.hashCode(key) ^ Objects.hashCode(value);
    }
    
    //为value设置新值，并返回oldValue
    public  final  V setValue(V newValue) {
        V oldValue  = value;
        value  = newValue;
        return  oldValue;
    }

    public  final  boolean  equals(Object o) {
        if  (o  == this) return  true;
        if  (o  instanceof  Map.Entry) {
            Map.Entry<?,?> e  = (Map.Entry<?,?>)o;
        if  (Objects.equals(key, e.getKey()) && 
            Objects.equals(value, e.getValue()))
            return  true;
        }
        return  false;
    }
}
```
<hr/>

- **红黑二叉树TreeNode<K,V>**

TreeNode<K,V> extends LinkedHashMap.Entry<K,V>

![mskt_04](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_04.png)

```
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 父节点
    TreeNode<K,V> left;  //左子树
    TreeNode<K,V> right; //右子树
    TreeNode<K,V> prev;    //上一个同级节点
    boolean red;//是否红色属性
   //构造函数
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
}
```

下面说下什么是红黑树，它一种特殊的二叉查找树。红黑树的每个节点上都有存储位表示节点的颜色，可以是红(Red)或黑(Black)。

红黑树的特征：

(1)根节点是黑节点</br>
(2)每个节点不是黑色就是红色</br>
(3)如果一个节点是红色的，则它的子节点必须是黑色的</br>
(4)每个为空(NIL或NULL)的叶子节点是黑色</br>
(5)没有一条路径会比其他路径长出俩倍。(平衡的二叉树)</br>
(6)左子树上所有结点的值均小于或等于它的根结点的值</br>
(7)右子树上所有结点的值均大于或等于它的根结点的值</br>
(8)左、右子树也分别为二叉排序树</br>
(9)从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点</br>
(10)从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点</br>

![mskt_05](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_05.png)

红黑树的应用比较广泛，主要是用它来存储有序的数据，它的时间复杂度是O(lgn)，效率非常之高。

<hr/>

- HashMap操作</br>
  
我们已经分析了HashMap中2种存储数据的结构。但什么时候去使用，如何使用这些数据结构存储数据，访问数据，删除数据呢？带着疑问继续看源码；默认构造HashMap的时候其实是没有table数组去分配空间，而是第一次put的时候才会分配，满足扩容条件resize方法.</br>

- (1)插入数据

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}
```
```java
/**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        /**先定义一个临时Node节点数组tab，p是单个节点的变量，n表示节点数组表的长度，i表示tab数组索引位置*/
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        /**如果table是刚初始化（null）,或者table的长度为0，则进行扩大数组tab的长度*/
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;/** 跟踪resize（）进去看源码*/
        if ((p = tab[i = (n - 1) & hash]) == null)
        	/** 通过hash值算出数组下标的位置，如果该位置为null，则创建一个节点*/
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            /** 如果数组中的要存的位置的该节点的hash值和key都分别相等
			则把要该位置的已经存在的值（old）的node节点引用赋值给一个临时的变量e*/
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)/**如果是一个红黑二叉树，存入该二叉树中并返回*/
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            	/** 如果是链表节点*/
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {/** 链表的尾部插入新的节点值*/
                        p.next = newNode(hash, key, value, null);
                        /** 如果节点数量达到链表转化为红黑树的阀值8，则转化为红黑二叉树*/
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;/** 循环终止*/
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    /**遍历链表，与前面的e = p.next组合，是可以遍历链表的条件*/
                    p = e;
                }
            }
            /**
             * 如果找到key值、hash值与插入元素相等的结点。
             * 新的value替换旧的value，并且返回旧值
             */
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;//表的结构被修改的次数增加
        if (++size > threshold)
            resize();/** 大于阀值来扩容*/
        afterNodeInsertion(evict);
        return null;
    }
```

针对以上put函数的源码，我们可以看出Hash表通过key的hashcode()得到的hash值，然后 hashCode%n,取余，就是数组中所在的位置索引。n为数组容量，这种方法用以保证计算出来的值都在数组size范围内。

hashMap支持null key，null value，key==null的时候，其hash值为0

<hr/>

- (2) resize 扩容
```java
 /**
     * 初始化或者对表的大小进行加倍，如果为null,则按初始容量目标分配。
     * 否则，因为我们使用的是2的幂次方扩充容量的，每个bin中的元素必须保持在相同的索引，
     * 或者在新表中以两倍偏移量移动。
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        /**对当前的table进行扩容*/
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        /** newCap:定义一个新的临时容量变量，newThr:定义个新的临时的threshold*/
        int newCap, newThr = 0;
        /**如果再扩充前的容量是大于0的*/
        if (oldCap > 0) {
            /**大于最大的MAXIMUM_CAPACITY*/
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;/**则把threshold = 21亿多（4个字节做大值）*/
                return oldTab;/**直接返回*/
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)/**如果位移1位，也就是翻倍（oldCap*2）,仍然没有大于MAXNUM*/
                newThr = oldThr << 1; /** newThr = oldCap*2 = threshold*2*/
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;/** AAA:初始化的capacity 被赋值给newCap*/
        else {               // zero initial threshold signifies using defaults
            /** 这种情况就是构造HashMap的时候，threshold初始化是0，第一次put数据resize，则我们就给
             * 给newCap赋值默认的初始化的容量16，newThr = 16*0.75
             * */
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        /**如果上面程序走的是AAA分支的话，那么newThr没有定位一个合适的值，此处就是给newThr赋合法值*/
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
          /** 创建节点数组，并且ru'guo赋值给table*/
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        /** 如果扩充前oldTab的值不是null值，那我们将进行把之前的旧值赋值到对应的newTab中*/
        if (oldTab != null) {
            /**遍历循环oldCap*/
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null; /** 如果j位置有节点元素，则取出该元素e，并且把数组中的该位置的node节点赋值null*/
                    if (e.next == null)/** 如果下个节点没有数据了（也就是链表的尾部），则把e,放在newTab位置为e.hash & (newCap - 1) 的地方*/
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)/** 如果是红黑树（点进split方法继续跟踪），重新映射，对红黑树进行split*/
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        /** 如果该节点是链表的存储
                         *  如果你对链表的结构，和java操作链表不会,那很难看懂
                         * */
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        /** 维持原来的链表的节点顺序进行rehash*/
                        do {
                            next = e.next; /** 当前节点指向的下一个节点地址*/
                            /** 原索引*/
                            if ((e.hash & oldCap) == 0) {/** e.hash & oldCap 这个计算的值，是节点数组的位置索引*/
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {/** 原所以加上oldCap*/
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        /** 原索引放在bucket里*/
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        /** 原索引+ oldCap放在bucket里*/
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                        /**
                         * 总结：
                         * 经过rehash之后，元素的位置要么在原来的位置，要么在原来的位置再移动2次幂。
                         * */
                    }
                }
            }
        }
 
        return newTab;
    }
```

总结：在jdk1.8中，resize方法是在hashmap中的键值对大于阀值时或者初始化时，就调用resize方法进行扩容，都是扩展2倍 扩展后Node对象的位置要么在原位置，要么移动到原偏移量两倍的位置。

(1) 我们在 HashMap 的构造函数中并没有看到为 table 数组分配空间的语句，因为这里采用了延迟加载的方式，直到第一次调用 put 方法时才会真正地分配数组空间。

(2) HashMap使用了基于（数组和单向链表）的哈希表的结构，数组中每一个元素都是一个链表，数组的初始化的长度是16，把数组中的每一格称为一个桶(bin或bucket)。当数组中已经被使用的桶的数量超过容量和加载因子的积，会进行扩容resize操作。

(3) 由于每一个桶（数组的单个元素）中都是一个单向链表，hash 相同的键值对都会作为一个节点被加入这个链表，当桶中键值对数量过多时会将桶中的单向链表转化为一个树。通过TREEIFY_THRESHOLD、UNTREEIFY_THRESHOLD和MIN_TREEIFY_CAPACITY来控制转换需要的阈值。

当然如果链表node的数量>8的时候，转化成红黑树。

如果红黑树的node的数量<6的时候，转化成链表。

(4) 如果只用单向链表的方式存储，那么查找数据的话处理哈希碰撞这种情况会对性能产生严重影响。时间复杂度为O(0)->O(n),而如果用红黑树的话时间复杂度也就只有O(log n).但是空间占用比较高。一般链表中的node的数量达到一定条件才会转换为红黑二叉树，这条件不会低。

<hr/>

- (3) get获取数据

![mskt_06](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_06.png)

```java
public V get(Object key) {
 
        Node<K,V> e;
 
        return (e = getNode(hash(key), key)) == null ? null : e.value;
 
}
 
final Node<K,V> getNode(int hash, Object key) {

    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    if ((tab = table) != null && (n = tab.length) > 0 &&

        (first = tab[(n - 1) & hash]) != null) {

        if (first.hash == hash && // always check first node

            ((k = first.key) == key || (key != null && key.equals(k))))

            return first;

        if ((e = first.next) != null) {

            if (first instanceof TreeNode)

                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            do {

                if (e.hash == hash &&

                    ((k = e.key) == key || (key != null && key.equals(k))))

                    return e;

            } while ((e = e.next) != null);

        }

    }

    return null;

}
```

以上源码可以意思大概是通过key算出，当初存的位置，然后从第一项开始查，如果没有查到，则通过f ((e = first.next) != null) 查询下一个节点。如果为红黑树节点，就在红黑树中找，否则就在链表中查。

<hr/>

- (4) remove删除
```java
final Node<K,V> removeNode(int hash, Object key, Object value,
 
                               boolean matchValue, boolean movable) {
 
        Node<K,V>[] tab; Node<K,V> p; int n, index;
 
        if ((tab = table) != null && (n = tab.length) > 0 &&
 
            (p = tab[index = (n - 1) & hash]) != null) {
 
            Node<K,V> node = null, e; K k; V v;
 
          //p指向bin中的第一个Entry
 
            //node指向目标Entry
 
            if (p.hash == hash &&
 
                ((k = p.key) == key || (key != null && key.equals(k))))
 
                node = p;
 
            else if ((e = p.next) != null) {
 
                if (p instanceof TreeNode)
 
                 //在红黑树中查找目标Entry
 
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
 
                else {
 
                    do {//在单链表中查找目标Entry
 
                        if (e.hash == hash &&
 
                            ((k = e.key) == key ||
 
                             (key != null && key.equals(k)))) {
 
                            node = e;
 
                            break;
 
                        }
 
                        p = e;
 
                    } while ((e = e.next) != null);
 
                }
 
            }
 
            if (node != null && (!matchValue || (v = node.value) == value ||
 
                                 (value != null && value.equals(v)))) {
 
               /**从红黑树中移除*/
 
                if (node instanceof TreeNode)
 
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
 
                /**移除单链表的第一个节点*/
 
                else if (node == p)
 
                    tab[index] = node.next;
 
                /**不是单链表的第一个节点*/
 
                else
 
                    p.next = node.next;
 
                ++modCount; /**增加修改计数器的次数*/
 
                --size; /**修改大小*/
 
                afterNodeRemoval(node); /**回调*/
 
                return node;
 
            }
 
        }
 
        return null;
}
```