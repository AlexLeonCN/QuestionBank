# SpringMVC的原理

## 1、Spring mvc介绍
SpringMVC框架是以请求为驱动，围绕Servlet设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是DispatcherServlet，它是一个Servlet，顶层是实现的Servlet接口。

## SpringMVC使用
需要在web.xml中配置DispatcherServlet。并且需要配置spring监听器ContextLoaderListener
```xml
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>        
        <servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                <!-- 如果不设置init-param标签，则必须在/WEB-INF/下创建xxx-servlet.xml文件，其中xxx是servlet-name中配置的名称。  -->
                <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring/springmvc-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

## 3、SpringMVC运行原理

![mskt_28](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_28.png)

![mskt_27](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_27.png)

流程说明：
- （1）客户端（浏览器）发送请求url，直接请求到DispatcherServlet。
- （2）DispatcherServlet根据请求信息调用HandlerMapping，解析请求对应的Handler, 并返回Controller的名字
- （3）解析到对应的Handler名字后，开始由HandlerAdapter适配器处理。
- （4）HandlerAdapter会根据名字来调用真正的处理器ControllerHandler开处理请求，并处理相应的业务逻辑。
- （5）处理器处理完业务后，会返回一个ModelAndView对象，Model是返回的数据对象，View是个逻辑上的View(View的名字)。
- （6）ViewResolver会根据逻辑View查找实际的View。
- （7）DispaterServlet把返回的Model传给View，进行渲染。
- （8）通过View返回给请求者（浏览器）