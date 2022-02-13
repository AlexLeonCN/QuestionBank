# 无框架下配置web.xml的主要配置内容是什么

一、
```xml
<servlet>
    <servlet-name>Hello</ servlet-name>//随便取
    <servlet-name>test.Hello</servlet-name>包名加类名
</servlet>
<servlet-mapping>
    <servlet-name>Hello</ servlet-name>//随便取，与上面的一致
    <url-pattern>/test/hello</servlet></url-pattern>//填写jsp中的跳转地址，/代表服务器的根目录
</ servlet-mapping>
```

二、web.xml用于配置Web应用的相关信息，如：监听器（listener）、过滤器（filter）、Servlet、相关参数、会话超时时间、安全验证方式、错误页面等，下面是一些开发中常见的配置：
- 1 配置Spring上下文加载监听器，加载Spring配置文件并创建IoC容器：
```java
  <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
 
  <listener>
     <listener-class>
       org.springframework.web.context.ContextLoaderListener
     </listener-class>
  </listener>
```
- 2 配置Spring的OpenSessionInView过滤器来解决延迟加载和Hibernate会话关闭的矛盾：
```java
  <filter>
      <filter-name>openSessionInView</filter-name>
      <filter-class>
         org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
      </filter-class>
  </filter>
 
  <filter-mapping>
      <filter-name>openSessionInView</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
```
- 3 配置会话超时时间为10分钟：
```java
  <session-config>
      <session-timeout>10</session-timeout>
  </session-config>
```
- 4 配置404和Exception的错误页面：
```java
  <error-page>
      <error-code>404</error-code>
      <location>/error.jsp</location>
  </error-page>
 
  <error-page>
      <exception-type>java.lang.Exception</exception-type>
      <location>/error.jsp</location>
  </error-page>
```
- 5 配置安全认证方式：
```java
  <security-constraint>
      <web-resource-collection>
          <web-resource-name>ProtectedArea</web-resource-name>
          <url-pattern>/admin/*</url-pattern>
          <http-method>GET</http-method>
          <http-method>POST</http-method>
      </web-resource-collection>
      <auth-constraint>
          <role-name>admin</role-name>
      </auth-constraint>
  </security-constraint>
 
  <login-config>
      <auth-method>BASIC</auth-method>
  </login-config>
 
  <security-role>
      <role-name>admin</role-name>
  </security-role>
```
**SSM框架中web.xml的配置内容**
```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">
	<display-name>CGB-SPRINGMVC-01</display-name>
 <!-- 配置前端控制器 -->
	<servlet>
		<servlet-name>frontController</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class><!--这里配置DispatcherServlet -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-config.xml</param-value><!--要加载这个xml -->
		</init-param>
   <!-- tomcat 启动时则初始化servlet,数字越小优先级越高 -->
		<load-on-startup>1</load-on-startup><!-- 这里是指需要servlet在tomcat容器启动的时候去加载 -->
	</servlet>
	<servlet-mapping>
		<servlet-name>frontController</servlet-name><!-- 上面和下面名字要求一致 -->
		<url-pattern>/</url-pattern><!-- 这里指的是处理哪些请求，/代表所有 -->
	</servlet-mapping>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>
</web-app>
````