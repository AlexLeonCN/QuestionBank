# session和cookie的区别和联系

## Cookie和Session之间的联系
session和cookie都是跟踪用户的整个会话

## Cookie和Session之间的区别
- 1.Cookie数据存放在客户的浏览器(本地),session数据放在服务器上
- 2.Cookie不如session安全，别人可以分析存放在本地的Cookie并进行Cookie欺骗,所以出于安全性的考虑应当使用Session
- 3.Session会在一定时间内保存在服务器上。当访问增多,会占用较多的服务器资源,所以出于性能考虑则应当使用cookie
- 4.session因为是保存在服务器上,所以不支持跨域的访问
>单个cookie保存的数据不能超过4k,很多浏览器都限制一个站点最多保存20个cookie.实际上为了性能考虑,不论是cookie还是session,其中的信息都应当短小精悍

## 什么是Cookie？
Cookie 是在HTTP协议下，服务器或脚本可以维护客户工作站上信息的一种方式。</br>
Cookie 是由 Web服务器保存在用户浏览器（客户端）上的小文本文件(内容通常经过加密)，它可以包含有关用户的信息。无论何时用户链接到服务器，Web站点都可以访问。</br>
Cookie 信息,可以看作是浏览器缓存.

## 什么是Session?
Session的定义很抽象,在不同的场合中session一词的含义也很不相同.</br>
它可以代表服务器与浏览器的一次会话过程,指从一个浏览器窗口打开到关闭的这个期间.</br>
也可以用于指一类用来在客户端与服务器之间保持状态的解决方案.

## Cookie机制
Cookie需要解决三个问题：`分发、内容和使用`

**cookie的分发**是通过扩展HTTP协议来实现的，服务器端通过在HTTP的响应由中加上一行特殊的指示以提示浏览器按照指示生成相应的cookie。

**cookie的内容**主要包括name(名字)、value(值)、maxAge(失效时间)、path(路径),domain(域)和secure
- **name**：cookie的名字，一旦创建，名称不可更改。
- **value**：cookie的值，如果值为Unicode字符，需要为字符编码。如果为二进制数据，则需要使用BASE64编码.
- **maxAge**：cookie失效时间，单位秒。
    - 如果为正数，则该cookie在maxAge后失效。
    - 如果为负数，该cookie为临时cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该cookie。
    - 如果为0，表示删除该cookie。
    - 默认为-1.
- **path**：该cookie的使用路径。
    - 如果设置为`"/sessionWeb/"`，则只有ContextPath为`“/sessionWeb/”`的程序可以访问该cookie。
    - 如果设置为`“/”`，则本域名下ContextPath都可以访问该cookie。
- **domain**:域.可以访问该Cookie的域名。
    - `第一个字符必须为"."`
    - 如果设置为".google.com",则所有以"google.com结尾的域名都可以访问该cookie".
    - 如果不设置,则为所有域名
- **secure**：该cookie是否仅被使用安全协议传输。

cookie的使用是由浏览器按照一定的原则在后台自动发送给服务器。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP请求头上发送给服务器.

## Session机制
Session机制是一种服务端的机制，服务器使用一种类似散列表的结构来保存信息。</br>
当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端里的请求里是否已包含了一个session标识--`sessionID`。</br>
如果已经包含一个sessionID，则说明以前已经为此客户端创建过session，服务器就按照sessionID把这个session检索出来使用（检索不到，可能会新建一个）。</br>
如果客户端请求不包含sessionID，则为此客户端创建一个session并且声称一个与此session相关联的sessionID，sessionID的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串(服务器会自动创建),这个sessionID将被在本次响应中返回给客户端保存。

## 保存sessionID的方式
1.使用Cookie.此时Cookie不再用作认证,只是作为载体,存储sessionID</br>
2.URL重写：因为cookie可以被人为禁止，所以为了保证sessionID能够被传递给服务器，使用URL重写也是一种解决方法。如
```jsp
href="<%=response.encodeURL("index.jsp") %>"
```
写好后URL是这样的
```jsp
href="index.jsp;jsessionid=0CCD096E7F8D97B0BE608AFDC3E1931E"
```
3.隐藏表单提交.将sessionId放在隐藏表单中,例如:
```jsp
<form name="testform" action="/xxx">
<input type="hidden" name="jsessionid" value="ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764">
<input type="text">
</form>
```
>实际上这几种方式最方便的还是cookie,其他两种方式都各有弊端:</br>URL重写的弊端是要对所有的URL都重写,非常繁琐.</br>表单隐藏字段的弊端是只能局限于表单提交,如果是单纯的文本链接则不能提供会话跟踪

![mskt_17](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_17.png)

Session的相关类库

上从图中可以看出从 request.getSession 中获取的 HttpSession 对象实际上是 StandardSession 对象的门面对象，这与前面的 Request 和 Servlet 是一样的原理。下图是 Session 工作的时序图：

![mskt_17](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_18.png)
