# 讲讲io里面的常见类，字节流，字符流，接口，实现类，方法阻塞的理解


![mskt_12](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_12.png)

首先什么是流：
>流（Stream）的概念来源于UNIX中的管道（pipe）概念，在unix中，管道是一条不间断的字节流，用来实现程序和进程间的通信，或者读写外围设备、外部文件等。流，必须有源端和目的端，可以是文件，内存或者网络等。流的创建是为了更方便的处理数据的输入输出。简单的来说：输入流就是从外部文件输入到内存，输出流主要是从内存输出到文件。我们用Eclipse开发小程序在控制台输入数据就属于输入流，即从控制台输入到内存。

其次对于字节流和字符流的区别：
>字节流处理单元为1个字节，操作字节和字节数组，而字符流处理的单元为2个字节的Unicode字符，分别操作字符、字符数组或字符串。</br>`程序中的输入输出都是以流的形式保存的，流中保存的实际上全都是字节文件。`</br>所有文件的储存是都是字节（byte）的储存，在磁盘上保留的并不是文件的字符而是先把字符编码成字节，再储存这些字节到磁盘。</br>在读取文件（特别是文本文件）时，也是一个字节一个字节地读取以形成字节序列。</br>那么既然磁盘存储都是按字节，内存中处理为何还需要字符流呢？</br>字符流是由Java虚拟机将字节转化为2个字节的Unicode字符为单位的字符而成的，所以它对多国语言支持性比较好！如果是音频文件、图片、歌曲，就用字节流好点，如果是关系到中文（文本）的，用字符流好点！

那开发中究竟用字节流好还是用字符流好呢？

>在所有的硬盘上保存文件或进行传输的时候都是以字节的方法进行的，包括图片也是按字节完成，而字符是只有在内存中才会形成的，所以使用字节的操作是最多的。我们建议尽量尝试使用字符流，一旦程序无法成功编译，就不得不使用面向字节的类库，即字节流。

上图中可以看到InputStream和OutputStream属于字节流，分别是输入字节流和输出字节流。而Reader和Writer属于字符流，分别对应输入字符流和输出字符流。

## 字节流

字节流主要是操作byte类型数据，以byte数组为准，主要操作类就是OutputStream、InputStream。两者都是抽象类，是字节输入、输出流的父类。以InputStream为例，最常用到的子类有：`FileInputStream，ByteArrayInputStream，ObjectInputStream，BufferedInputStream、DataInputStream、PushbackInputStream`。

其中前三类可以理解为针对流中不同的内容分别实现了类，这点可以通过查看各自类的构造函数来验证，比如说FileInputStream的一个构造方法为：`FileInputStream(File file) `，即通过打开一个到实际文件的连接来创建一个FileInputStream。ByteArrayInputStream类的构造方法`ByteArrayInputStream(byte[] buf) `的参数为byte[]数组。

而后三类属于FilterInputStream，这些类构造方法的参数为InputStream的实例，比如说`BufferedInputStream(InputStream in)` 。然而InputStream是抽象类，无法直接实例化，所以我们可以放前三类的实例对象进去。

也许下面这段话可以帮助理解：

>BufferedInputStream和InputStream类都是字节流，而更确切地说，BufferedInputStream是缓冲字节流。</br>InputStream是不带缓冲的操作，每读一个字节就要写入一个字节，由于涉及磁盘的IO操作相比内存的操作要慢很多，所以不带缓冲的流效率很低。</br>BufferedInputStream带缓冲的流，可以一次读很多字节，但不向磁盘中写入，只是先放到内存里。等凑够了缓冲区大小的时候一次性写入磁盘，这种方式可以减少磁盘操作次数，速度就会提高很多！这是两者的区别。

为什么说字节流主要是操作byte类型的数组呢？请看以下示例：

```java
import java.io.BufferedInputStream;  
import java.io.File;  
import java.io.FileInputStream;  
import java.io.FileNotFoundException;  
import java.io.IOException;  
public class BufferedInputStreamTest {  
      public static void main(String[] args) throws IOException {  
        File file = new File("c:\\mm.txt");  
        FileInputStream fis = new FileInputStream(file);  
        BufferedInputStream bis = new BufferedInputStream(fis);  
          
        byte[] contents = new byte[1024];  
        int byteRead = 0;  
        String strFileContents;  
         
        try {  
            while((byteRead = bis.read(contents)) != -1){  
                strFileContents = new String(contents,0,byteRead);  
                System.out.println(strFileContents);  
            }  
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        bis.close();  
    }  
} 
```
- 可见要想读取InputStream实现类中的数据，需要用到byte[] 数组，然后将其转化为字符串并输出。
- 同时以上代码展示了BufferedInputStream的构造方法的一种参数形式。
- OutputStream同InputStream是一个道理，不再赘述。

## 字符流

在程序中一个字符等于两个字节，java提供了Reader、Writer两个专门操作字符流的类。两者同样是抽象类。已Reader为例，其常用的子类有：`BufferedReader,CharArrayReader,FilterReader, InputStreamReader,PipedReader,StringReader `。

- 其中**FilterReader**是抽象类，其实例化的类为PushbackReader。
- 文章开头图中的**ByteArrayReader**其实应为此处的CharArrayReader，其构造函数的参数要接收字符数组。
- **PipedReader**是管道输入流，需要和PipedWriter合起来用。
- **StringReader**是源为一个字符串的字符流，即其构造函数的参数要接收字符串。
- **InputStreamReader** 是字节流通向字符流的桥梁：它使用指定的 charset读取字节并将其解码为字符。
>`注意：**FileReader**是**InputStreamReader**的子类！`</br>**InputStreamReader**构造方法的参数是什么呢？是**InputStream**！<br/>`也就是说可以将字节输入流封装到InputStreamReader中，实现字节流到字符流的转换！`</br>

另外,InputStreamReader和OutputStreamWriter这两个类是字节流和字符流之间的适配器类，它们承担编码转换的任务。
- **BufferedReader**从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。 其构造方法的参数为Reader子类。
>通常，Reader 所作的每个读取请求都会导致对底层字符或字节流进行相应的读取请求。因此，建议用 BufferedReader 包装所有其 read() 操作可能开销很高的 Reader（如 FileReader 和 InputStreamReader）。例如:
>```java
>BufferedReader in 
>       = new BufferedReader(new FileReader("foo.in"));
>```

为了进一步说明字节流和字符流所处理的对象不同，我们来看一下BufferedInputStream和BufferedReader的主要read方法。

首先是BufferedInputStream：

返回值 | 方法名 | 作用
---|---|---
int | `read()` | 参见 InputStream 的 read 方法的常规协定。
int | `read(byte[] b, int off, int len)`| 从此字节输入流中给定偏移量处开始将各字节读取到指定的 byte 数组中。

然后是BufferedReader：


返回值 | 方法名 | 作用
---|---|---
int | `read()` |  读取单个字符
int | `read(char[] cbuf, int off, int len)` | 将字符读入数组的某一部分
String | `readLine()` | 读取一个文本行