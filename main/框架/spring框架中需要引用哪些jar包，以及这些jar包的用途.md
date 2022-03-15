# spring框架中需要引用哪些jar包，以及这些jar包的用途

spring.jar是包含完整发布的单个jar包。除了spring.jar文件外还包含13个独立的jar包，各自对应spring不同的组件，在使用时可以根据实际情况选择需要的jar包，不必引入整个spring.jar包中所有的文件

序号 | jar包 | 作用
---|---|---
1 | spring-core.jar | 包含spring框架基本的核心工具类，其他组件都要使用这个包里面的类，是其他组件的核心
2 | spring-bean.jar | 是所有的应用都要用到的，包含访问配置文件、创建和管理bean以及进行IoC和DI操作所需的相关类
3 | spring-aop.jar | 包含使用AOP特性时所需的类
4 | spring-context.jar | 为spring核心提供了大量扩展
5 | spring-dao.jar | 包含spring DAO、spring Transaction进行数据访问的所有类
6 | spring-hibernate.jar | 包含spring对hibernate 2以及hibernate 3进行封装的所有类
7 | spring-jdbc.jar | 包含spring对JDBC数据库访问进行封装的所有类
8 | spring-orm.jar | 包含多DAO特性集进行了扩展
9 | spring-remoting.jar | 包含支持EJB、JMS、远程调用Remoting方面的类
10 | spring-support.jar | 包含支持缓存Cache、JAC、JMX、邮件服务、任务计划Scheduling方面的类
11 | spring-web.jar | 包含web开发时，用到spring框架时所需的核心类
12 | spring-webmvc.jar | baohan Spring MVC框架相关的所有类
13 | spring-mock.jar | 包含spring一整套mock类来辅助应用的测试