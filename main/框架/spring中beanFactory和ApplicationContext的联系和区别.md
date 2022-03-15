# spring中beanFactory和ApplicationContext的联系和区别

![mskt_29](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_29.png)

**BeanFactory**是Spring中最基本、通用的工厂，这里的工厂，其不仅仅是构造实例，它可以创建并管理各种类的对象，即是Spring IOC容器体系结构的基本接口，其与其子接口便构成了Spring IOC容器的体系结构(如上图：IOC容器主要接口关系图)。</br>
所以不要把它理解成常规意义的简单工厂，也可以把它称为Sring容器。

**ApplicationContext**也是一个Spring容器,它是由BeanFactory接口派生而来，因而提供BeanFactory所有的功能，此外它还扩展了BeanFactory的功能，提供了更多的高级功能：
- MessageSource, 提供国际化的消息访问
- 访问资源，如URL和文件（ResourceLoader）
- 事件传播
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的**web层**。
- 消息发送、响应机制（ApplicationEventPublisher）
- AOP（拦截器）

## 联系
所以联系上它们本身都是Spring提供的接口，它们都和其子接口构成了Spring IOC的容器，我们一般都称他们为Spring容器。其中BeanFactory是Spring框架的基础设施，面向Spring本身，其提供基本的功能，如：getBean()，；</br>
ApplicationContext是建立在它之上，面向使用者，扩展了更多面向应用的功能，更易于创建实际应用，实际也基本使用它。

## 区别
- 由上述可知他们提供的功能是有不同的，体系结构、用途等有所不同。</br>
- 另外他们在初始化时有一个重大的区别：`对象实例化是否延迟`
    - BeanFactroy初始化容器时，并未初始化Bean，直到第一次在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化(延迟加载)，这样的话，我们就不能在初始化容器时发现一些存在的Spring的配置问题。
    - ApplicationContext则相反，它是在容器启动时，就一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误。

## 延迟实例化的优点：（BeanFactory）
应用启动的时候占用资源很少；对资源要求较高的应用，比较有优势；

## 不延迟实例化的优点： （ApplicationContext）
- 1.所有的Bean在启动的时候都加载，系统运行的速度快；
- 2.在启动的时候所有的Bean都加载了，我们就能在系统启动的时候，尽早的发现系统中的配置问题
- 3.建议web应用，在启动的时候就把所有的Bean都加载了。（把费时的操作放到系统启动中完成） 