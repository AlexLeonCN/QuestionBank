# JSP九大内置对象 和 四个域对象

**Application > Session 会话> Request > Page**</br>
application作用域范围:一个应用服务器范围内有效。就是服务器启动到关闭的整段时间</br>
Session作用域范围：一次会话范围内有效。同一浏览器对服务器进行多次访问，在这多次访问之间传递信息，就是session作用域的体现。</br>
Request作用域范围：在一个服务器请求范围内有效</br>
pagez作用域范围：在一个页面范围内有效，通过pageContext对象访问</br>

名称 | 对象 | 类型(转译后对应Servlet) | 作用域
---|---|---|---
request | 请求对象 | javax.servlet.ServletRequest | Request
response | 响应对象 | javax.servlet.SrvletResponse | Page
pageContext | 页面上下文对象 | javax.servlet.jsp.PageContext | Page
session | 会话对象 | javax.servlet.http.HttpSession | Session
application | 应用程序对象 | javax.servlet.ServletContext | Application
out | 输出对象 | javax.servlet.jsp.JspWriter | Page
config | 配置对象 | javax.servlet.ServletConfig | Page
page | 页面对象 | javax.lang.Object | Page
exception | 异常对象 | javax.lang.Throwable | Page

![mskt_19](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_19.png)