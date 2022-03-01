# 说一下Java内存模型和GC算法

我们用的JVM是hotspot虚拟机，是server VM版，server版专用于服务端处理并发，还有个client版，一般用于手机端，占内存高。

## JVM内存结构

JVM hotspot JDK8 可分为三部分：
- 类加载 ClassLoader
- 运行时数据区 JavaRuntimeDataArea
- 执行引擎系统 JVM Excution Engine System

![mskt_21](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_21.png)

### 1.类加载
程序在执行的时候，首先会编译成.class这样的文件，而这些文件在运行的时候会被JVM的类加载系统(JVM ClassLoading System)加载到内存里面，这些文件执行的时候可能是多个(有很多个xxx.class文件)。

加载器 加载过程分为
- 加载(loading)
- 连接(linking) 其中连接过程又分为：
    - 验证(verification)
    - 准备(preparation)
    - 解析(Resolution)
- 初始化(initialization)

![mskt_22](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_22.png)

### 2.运行时数据区 Java Runtime Data Area

运行时数据区

![001](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/JVM/001.png)

底层通过IO流将.class读到内存里，然后会被放在JVM 方法区，方法区是内存中的一块区域，这块区域可以被分成两大部分，一个是 `线程共享区(堆，和方法区) `，一块是 `线程私有区(栈) `。

方法区仅仅是逻辑上的一个概念，在不同的JVM里面有不同的呈现，在JDK8之前方法区叫 `持久代(永久带) `，JDK8之后永久带被 `元空间meta space `替代，不管是元空间还是永久带，他们都是方法区。

操作系统OS为JVM分配内存区域，JVM再给堆和元数据区分配。但是这样会导致JVM被分配的内存空间不足或者浪费，现在提出一个新的策略：JVM直接访问外部的内存，将元数据区放到外部，不占JVM的内存空间，直接由操作系统OS独立分配。

![mskt_23](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_23.png)

JVM里面有一个垃圾回收GC回收系统，GC的作用就是对堆区进行内存回收 。

#### 堆
堆里面又分成若干个区域，老年代，年轻代。年轻代里面又分为新生代(Eden伊甸园区)，和幸存区。

#### 分代回收算法
幸存区一般有s0,s1两个，举个例子：
>一开始两个幸存区都是空的。<br>项目运行的时候不停的在创建对象，当eden区放满了会触发第一次GC，回收后幸存下来的对象被放到s0区域。<br/>随着业务的执行，还在创建对象放在eden区，当eden又放满后，再次触发GC, 回收eden和s0，幸存下来的对象，再放到s1区域，这个时候s0会被清空。<br/>下次再触发的时候，幸存对象就会放到s0里面，这样反复循环下去。<br/>当幸存的对象达到一定年龄后，比如说经过8次GC还没被回收的对象，就会变成老对象，被放到老年代。当老年代满了，Eden区也满的时候，就会触发大GC，既回收年轻代，也回收老年代。这样回收有一定内存资源浪费，有一块内存区域永远是空的。这就是分代回收算法的设计思想。

大部分对象用完以后会释放内存，所以大部分对象刚创建都是放在年轻代，随着反复回收幸存下来的对象会被放在老年代。
>注：无论是堆还是方法区，都放在线程共享区。在线程共享区就要考虑线程安全问题。

#### 栈
栈结构形象点理解起来像是弹夹，下面封死的，入栈和出栈同一个口，所以会先进后出。 `即栈是一种只能从表的一端存取数据且遵循 "先进后出" 原则的线性存储结构。 `

栈又叫做方法栈(线程栈)，有两部分，一部分叫 `java方法栈(java method stack) `，还有一个叫  `本地方法栈(native method stack) ` 。本地方法栈一般使用c语言来编写的。Object类里面就有很多本地方法，用native关键字修饰的。

因为栈属于线程私有区，当线程创建的时候栈创建，当线程结束的时候栈结束。</br>
当线程创建的时候就会创建这个方法栈(线程栈)，用于存储这个线程下面运行的方法。</br>
方法栈(线程栈)里面还存储了 `栈帧(stack frame) `，栈帧里面存储的是方法信息：返回值类型，返回值，方法运行时的局部变量。</br>
方法栈(线程栈)里还存储了 `程序计数器Program counter `，程序计数器的作用：记录CPU在执行线程的位置，切换回来的时候还能从上次执行到的位置开始执行。

### 3.JVM执行引擎系统 JVM Excution Engine System
执行时数据区Runtime data area中的内存数据要交给 JVM执行引擎系统Excution Engine System这个对象来执行。</br>
>【扩展】mySql也有建表引擎，建表的语句中会出现引擎ENGINE=InnoDB, 假如没有指定引擎，默认的引擎就是InnoDB，这个数据库引擎决定了数据的存储方式以及是否支持事务，InnoDB引擎支持事务处理，而还有一种MYISAM引擎是不支持事务的。

JVM执行系统引擎里面又分为  `解释执行 ` 和  `编译执行 `</br>
**解释执行** ：java文件编译为.class字节码文件，解释器 Interpeter 将字节码数据解释为计算机能懂的二进制语言，这个叫做解释执行， 每次使用都需要将数据解释，如果代码重复度高，性能较低。Java中是解释一行执行一行，不是全部解释完再执行代码。</br>
**JIT 编译执行 JIT Compiler**   特点：可以对同一代码反复执行，JIT对象中含有一个cache。如果是热点代码(hot code重复度比较高)，可以先将此代码解释后，存入cache内存中，以后使用直接从cache中取就可以。
现在的JVM同时含有解释执行，编译执行两种执行方式。
>【扩展】相同地，在mySql可以使用预编译的sql语句来提高性能<br/>比如insert into a values(?,?,?)，为什么在JDBC里面推荐使用这个预编译格式的sql语句呢？<br/>应用程序和数据库交互的时候使用的是sql语句，sql语句是从应用程序端发送到数据库端，JDBC里面有个传送器叫statement, statement负责将sql语句从应用程序端发送到数据库端，数据库收到后要存这个sql语句，如果是非预编译的普通sql语句，那么数据库端需要反复存取，但是我们如果传insert into a values(?,?,?)这种sql语句，运行的时候给不同的值，数据库端只需要存一次。

![mskt_20_GC](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_20_GC%E5%9B%9E%E6%94%B6.png)