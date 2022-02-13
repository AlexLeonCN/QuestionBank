# 解释一下你对servlet的理解

## 你对Servlet的理解？

Servlet（Servlet Applet），全称Java Servlet,是用Java编写的`服务器端程序`<br/>
而这些Servlet都要实现Servlet这个接口。<br/>
其主要功能在于交互式的浏览和修改数据，生成动态Web内容。Servlet运行于支持Java的应用服务器中。

HttpServlet 重写doGet 和 doPost 方法或者你也可以重写service方法完成对get和post请求响应。

## Servlet的通俗理解？
Servlet是一个运行了面向请求/ 响应服务器中的网络模块。</br>
请求是客户的一个调用，可能是远程的。</br>
请求包含了客户要发送给服务器的数据。

响应是服务器返回给客户的响应该请求的数据。</br>
Servlet是一个JAVA对象，他以请求为输入，分析其数据，执行一些逻辑运算，并给客户发回一个响应。</br>
另一方面，Servlet作为驻留在服务器端HTTP明白的中间层，它们知道怎样在HTTP中通过RMI或IIOP在EJB和客户之间进行通信。

**Servlet体系结构**

![mskt_14](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_14.png)

Servlet的框架是由两个Java包组成的：`javax.servlet`与`javax.servlet.http`。</br>
在javax.servlet包中定义了所有的Servlet类都必须实现或者扩展的通用接口和类。</br>
在javax.servlet.http包中定义了采用Http协议通信的HttpServlet类。</br>
Servlet的框架的核心是javax.servlet.Servlet接口，所有的Servlet都必须实现这个接口。

![mskt_15](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_15.png)

在Servlet接口中定义了5个方法：
- **init(ServletConfig)方法**：负责初始化Servlet对象，在Servlet的生命周期中，该方法执行一次；该方法执行在单线程的环境下，因此开发者不用考虑线程安全的问题；
- **service(ServletRequest req,ServletResponse res)** 方法：负责响应客户的请求；为了提高效率，Servlet规范要求一个Servlet实例必须能够同时服务于多个客户端请求，即service()方法运行在多线程的环境下，Servlet开发者必须保证该方法的线程安全性；
- **destroy()** 方法：当Servlet对象退出生命周期时，负责释放占用的资源；
- **getServletInfo**：就是字面意思，返回Servlet的描述；
- **getServletConfig**：这个方法返回由Servlet容器传给init方法的ServletConfig。

## Servlet工作原理
当Web服务器接收到一个HTTP请求时，它会先判断请求内容——如果是静态网页数据，Web服务器将会自行处理，然后产生响应信息；如果牵涉到动态数据，Web服务器会将请求转交给Servlet容器。此时Servlet容器会找到对应的处理该请求的Servlet实例来处理，结果会送回Web服务器，再由Web服务器传回用户端。

针对同一个Servlet，Servlet容器会在第一次收到http请求时建立一个Servlet实例，然后启动一个线程。第二次收到http请求时，Servlet容器无须建立相同的Servlet实例，而是启动第二个线程来服务客户端请求。所以多线程方式不但可以提高Web应用程序的执行效率，也可以降低Web服务器的系统负担。

![mskt_13](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_13.png)

>Web Client 向Servlet容器（Tomcat）发出Http请求；</br>
>Servlet容器接收Web Client的请求；</br>
>Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中；</br>
>Servlet容器创建一个HttpResponse对象；</br>
>Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet对象；</br>
HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息；</br>
HttpServlet调用HttpResponse对象的有关方法，生成响应数据；</br>
Servlet容器把HttpServlet的响应结果传给Web Client；

![mskt_16](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_16.png)