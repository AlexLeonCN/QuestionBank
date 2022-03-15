# jsp和servlet的区别是什么

- jsp经编译后就变成了Servlet.(JSP的本质就是Servlet，JVM只能识别java的类，不能识别JSP的代码,Web容器将JSP的代码编译成JVM能够识别的java类)</br>
- jsp更擅长表现于页面视图显示,servlet更擅长于逻辑控制.</br>
- Servlet中没有内置对象，Jsp中的内置对象都是必须通过HttpServletRequest对象，HttpServletResponse对象以及HttpServlet对象得到.</br>
- Jsp是Servlet的一种简化，使用Jsp只需要完成程序员需要输出到客户端的内容- Jsp中的Java脚本如何镶嵌到一个类中？由Jsp容器完成。</br>
- Servlet则是个完整的Java类，这个类的Service方法用于生成对客户端的响应
- Servlet完全是JAVA程序代码构成，擅长于流程控制和事务处理而通过Servlet
  来生成动态网页;</br>
- JSP由HTML代码和JSP标签构成，可以方便地编写动态网页。因此在实际应用中采用Servlet来控制业务流程,而采用JSP来生成动态网页。</br>
- JSP是Servlet技术的扩展，本质上就是Servlet的简易方式，但是两者的创建方式不一样.</br>
- Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML里分离开来。</br>