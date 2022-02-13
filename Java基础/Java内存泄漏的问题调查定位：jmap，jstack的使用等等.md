# Java内存泄漏的问题调查定位：jmap，jstack的使用等等

jstack是用于调试线程间的关系，解决阻塞，锁等问题</br>
jmap用于输出对象，适用于解决对象内存溢出等问题

jstack生成当前线程dump信息，主要用于查看某个线程的堆栈信息。</br>
jmap生成堆dump信息, 用来查看堆内存使用状况，一般结合jhat使用</br>
jstat 查看虚拟机的一些统计信息，比如gc信息</br>
jps查看正在运行的虚拟机进程



**jstack主要用来查看某个Java进程内的线程堆栈信息**。语法格式如下：
- jstack [option] pid
- jstack [option] executable core
- jstack [option] [server-id@]remote-hostname-or-ip

jstack使用实例步骤
- 1.先找出Java进程ID，我部署在服务器上的Java应用名称为mrf-center,得到进程ID为21711
```
root@ubuntu:/# ps -ef | grep mrf-center | grep -v grep
root     21711     1  1 14:47 pts/3    00:02:10 java -jar mrf-center.jar
```
- 2.找出该进程内最耗费CPU的线程，可以使用</br>`ps -Lfp pid`</br>或者`ps -mp pid -o THREAD, tid, time`</br>或者`top -Hp pid`,输出如下:

![mskt_09](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_09.png)

- 3.TIME+列就是各个Java线程耗费的CPU时间，最长的是线程ID为21742的线程，使用</br>`printf "%x\n" 21742`</br>得到21742的十六进制值为54ee
- 4.使用jstack来输出进程21742的堆栈信息，然后根据线程ID的十六进制值grep
```
root@ubuntu:/# jstack 21711 | grep 54ee
"PollIntervalRetrySchedulerThread" prio=10 tid=0x00007f950043e000 nid=0x54ee in Object.wait() [0x00007f94c6eda000]
```
可以看到CPU消耗在PollIntervalRetrySchedulerThread这个类的Object.wait(),根据这个去定位代码。

jmap(Memory Map)和jhat(Java Heap Analysis Tool)</br>
**jmap用来查看堆内存使用状况，一般结合jhat使用**</br>
jmap语法格式如下：
```
jmap [option] pid
jmap [option] executable core
jmap [option] [server-id@]remote-hostname-or-ip
```
- 1.如果运行在64位JVM上，可能需要指定-J-d64命令选项参数</br>
  打印进程的类加载器和类加载器加载的持久代对象信息，输出：类加载器名称、对象是否存活（不可靠）、对象地址、父类加载器、已加载的类大小等信息.
```
jmap -permstat pid
```
- 2.使用jmap -heap pid查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况
```
jmap -heap 21711
```
- 3. 使用jmap -histo[:live] pid查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象
```
jmap -histo:live 21711 | more
```
- 4.用jmap把进程内存使用情况dump到文件中，再用jhat分析查看,jmap进行dump命令格式为：`jmap -dump:format=b,file=dumpFileName`
```
jmap -dump:format=b,file=/tmp/dump.dat 21711
```
```
jhat -port 9998 /tmp/dump.dat
```
然后就可以在浏览器中输入主机地址:9998查看了