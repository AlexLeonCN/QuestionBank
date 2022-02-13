# Redis存储的数据类型

Redis（remote dictionary server）是一个基于KEY-VALUE的高性能的 存储系统，通过提供多种键值数据类型来适应不同场景下的缓存与存储需求 。

redis是以字典结构存储数据的容器，并允许其他应用通过TCP协议读写字典中的内容，一个redis实例可以提供多个用来存储数据的字典，可以指定把相应的数据存储到哪个类型的字典里面。

Redis比memached提供了更丰富的数据结构，有五种数据结构：
- string字符串类型
- list列表
- hash
- set集合类型（set）
- sorted-set有序集合类型
  ![mskt_42](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_42.png)

**1.字符串类型：**

可以存储任何形式的字符串，包括二进制类型的数据，session,cookie,图片等，一个字符类型键允许存储的最大容量是512M 。

string类型对应的是key和value，string类型默认支持三种数据格式：字符串，整数，浮点

使用场景：session共享，短信验证码等，在多台Web服务器之间想完成一个session共享的话，有几种方式，session复制，cookie存储，把所有的session存储到redis里面(把所有session序列化以后存储到redis的字符串里面）。

**2.List**

使用场景：实现分布式队列，栈

**3.hash类型**

![mskt_43](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_43.png)
使用场景：hash的存储方式类似于关系型数据库，可以存储一些对象形式的信息

**4.集合Set**

集合类型中，每个元素都是不同的，也就是不能有重复数据，同时集合类型中的数据是无序的。一个集合类型键可 以存储至多232-1个 。集合类型和列表类型的最大的区别是有序性和唯一性 集合类型的常用操作是向集合中加入或删除元素、判断某个元素是否存在。由于集合类型在redis内部是使用的值 为空的散列表(hash table)，所以这些操作的时间复杂度都是O(1).
Set在的底层数据结构以intset或者hashtable来存储。当set中只包含整数型的元素时，采用intset来存储，否则， 采用hashtable存储，但是对于set来说，该hashtable的value值用于为NULL。通过key来存储元素
- 使用场景：去重，找标签，差集

**5.有序集合**

有序集合类型，和集合类型的区别就是多了有序的功能
在集合类型的基础上，有序集合类型为集合中的每个元素都关联了一个分数，这使得我们不仅可以完成插入、删除 和判断元素是否存在等集合类型支持的操作，还能获得分数最高(或最低)的前N个元素、获得指定分数范围内的元 素等与分数有关的操作。虽然集合中每个元素都是不同的，但是他们的分数却可以相同 。
zset类型的数据结构就比较复杂一点，内部是以ziplist或者skiplist+hashtable来实现，这里面最核心的一个结构就 是skiplist，也就是跳跃表 。