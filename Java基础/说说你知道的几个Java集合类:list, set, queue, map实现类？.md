## 说说你知道的几个Java集合类:list, set, queue, map实现类？

![mskt_07](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_07.png)

HashSet, linkedHashSet，HashMap,linkedHashMap：都是基于散列表实现的，即链表数组。

LinkedHashMap继承了HashMap，并在node上多加了两个指针，来维护每个node之间的顺序，原有的hashmap的结构是没变的。

HashSet只是封装下HashMap，实际就是一个hashMap，只是这个map只能对键进行操作，set里的元素被当做是Map的key来存的，map的value存的是一个static final new object（），循环这个set其实就是取得HashMap的keySet来循环。

LinkedHashSet继承了HashSet，调用的是HashSet里构造linkedHashMap的构造方法，实际就是个linkedHashMap，然后只能对map的key进行操作。

TreeSet，TreeMap: 都是基于红黑树实现的，特点是所得到的结果是经过排序的，次序由元素实现的Comparable或者Comparator决定。将元素加到TreeSet比HashSet慢，不过TreeSet是排序的。

PriorityQueue:是接口Queue的实现。 使用堆数据结构，堆是一个可以自我调整的二叉树，对树执行添加和删除操作，可以让最小的元素移动到根，而不必花费时间对元素进行排序。优先级队列可以按任意的顺序插入元素，却总是按照排序的顺序进行检索，也就是无论何时调用remove方法，总会获得当前优先级队列中最小的元素（即优先级最高的元素，习惯上将1设为最高优先级）。默认是按元素的自然顺序排序，如存储Integer，char，String类型的元素，也可以通过自定义Comparator来排序。但是与TreeSet不同，如果仅迭代这些元素，是不会对元素进行排序的，PriorityQueue可以确保调用peek，poll，remove方法时，获取的元素是队列中优先级最高的元素。

DelayQueue：无界阻塞队列，用于放置实现了Delayed接口的对象，其中对象只能在到期时才能从队列中取出。DelayQueue其实就是在每次往优先级队列中添加元素,然后以元素的delay/过期值作为排序的因素,以此来达到先过期的元素会拍在队首,每次从队列里取出来都是最先要过期的元素。