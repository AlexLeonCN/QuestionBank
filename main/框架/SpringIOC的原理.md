# Spring IOC的原理，底层如何管理对象的

IOC 容器初始化过程分析.

![mskt_37](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_37.png)

创建对象的时候，ApplicationContext为对象提供一个存储和应用的环境，这种ApplicationContext的实现由两种方式， (Xml)ClassPathXmlApplicationContext和(注解)AnnotationConfigApplicationContext。 Xml和注解都是描述数据的数据，ClassPathXmlApplicationContext正在逐步成为历史，而使用注解方式的越来越广泛，注解方式更加简单，更加灵活，可配置性高。

注：Mybatis中的工厂是SqlSessionFactory他是创建会话的对象, Spring中的对象是BeanFactory

![mskt_38](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_38.png)

通过配置文件的描述再容器中创建对象，外界需要的时候通过getBean方法从容器里获取对象。

有xml方式和注解方式:ClassPathXmlApplicationContext处理spring-configs.xml，AnnotationConfigApplicationContext处理SpringConfig.class。

每一个类经过包扫描解析后都会封装成IOC容器中的BeanDefinition对象， 封装的是类的配置信息。

Map集合中，String就是类的全名，BeanDefinition就是封装的类的配置信息。

基于这个配置信息就可以创建Bean对象，Bean对象直接还有可能有依赖关系，创建好Bean对象会放到BeanPool(Bean池)中去，这也是一种单例设计，保证Bean的唯一性来优化内存。

`所以IOC中有两大容器，一个是存放配置信息的，另一个是存放基于配置信息创建好的Bean实例。两个容器都会交给BeanFactory，getBean去池中取，池中没有就基于原材料去创建再放池里。`

Bean的特性：
- 1)Scope 作用域(Singleton protontype)
- 2)LifiCycle 生命周期
    - Spring启动的时候ApplicationContext对象会把所有Singleton单例对象创建，ProtonType只有在getBean的时候才会被创建，当不再需要Bean对象的时候会触发Destory清楚Bean对象。
- 3)延迟加载 @Lazy

注意：
- 1)spring ios容器包含两部分，一部分时存放原材料的容器，一部分是基于原材料创建的bean对象的容器 BeanPool
- 2)生命周期@PostConstruct @PreDestroy，和延迟加载lazy特性 只对singleton有效，因为`protontype对象默认在getBean的时候才初始化创建`，并且不会放到BeanPool中
- 3)ApplicationContext实现的默认行为就是在启动时将容器中所有singleton bean提前进行实例化（也就是依赖注入）。如果添加延迟，容器启动时不实例化bean，而是调用getBean方法时才实例化的。
- 4)指定包(如ComponentScan(""com.spring))下面有这样的注解描述的对象才会交给spring管理 @Component @Service @Controller @Repository @Bean ...