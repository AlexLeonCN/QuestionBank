# Spring解决循环依赖问题

## 什么是循环依赖

其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环
如图：

![001](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/001.png?x-oss-process=image/resize,w_250)

![002](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/002.png?x-oss-process=image/resize,w_250)

如何理解“依赖”呢，在Spring中有：

- 构造器循环依赖
- field属性注入循环依赖

构造器循环依赖 - 项目启动失败（发现一个circle）(除非使用@Lazy懒加载，懒加载会注入一个代理对象)
```java
@Service
public class A {  
    public A(B b) {  }
}
 
@Service
public class B {  
    public B(C c) {  
    }
}
 
@Service
public class C {  
    public C(A a) {  }
}
```

属性注入循环依赖(默认是singleton) - 项目启动成功
```java
@Service
public class A1 {  
    @Autowired  
    private B1 b1;
}
 
@Service
public class B1 {  
    @Autowired  
    public C1 c1;
}
 
@Service
public class C1 {  
    @Autowired  public A1 a1;
}
```

field属性注入循环依赖（prototype）
```java
@Service
@Scope("prototype")
public class A1 {  
    @Autowired  
    private B1 b1;
}
 
@Service
@Scope("prototype")
public class B1 {  
    @Autowired  
    public C1 c1;
}
 
@Service
@Scope("prototype")
public class C1 {  
    @Autowired  public A1 a1;
}
```

## 什么情况下循环依赖可以被处理

在回答这个问题之前首先要明确一点，Spring解决循环依赖是有前置条件的

- 出现循环依赖的Bean必须要是单例
- 依赖注入的方式不能全是构造器注入的方式（很多博客上说，只能解决setter方法的循环依赖，这是错误的）

## Spring是如何解决的循环依赖

关于循环依赖的解决方式应该要分两种情况来讨论

- 简单的循环依赖（没有AOP）
- 结合了AOP的循环依赖

### 简单的循环依赖（没有AOP）

我们先来分析一个最简单的例子
```java
@Component
public class A {
    // A中注入了B
	@Autowired
	private B b;
}

@Component
public class B {
    // B中也注入了A
	@Autowired
	private A a;
}
```
Spring在创建Bean的时候默认是按照自然排序来进行创建的，所以第一步Spring会去创建A。

与此同时，我们应该知道，Spring在创建Bean的过程中分为三步

- 实例化，对应方法：`AbstractAutowireCapableBeanFactory中的createBeanInstance方法`

- 属性注入，对应方法：`AbstractAutowireCapableBeanFactory的populateBean方法`

- 初始化，对应方法：`AbstractAutowireCapableBeanFactory的initializeBean`

实例化，简单理解就是new了一个对象<br/>
属性注入，为实例化中new出来的对象填充属性<br/>
初始化，执行aware接口中的方法，初始化方法，完成AOP代理<br/>

![003](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/003.png)

创建A的过程实际上就是调用getBean方法，这个方法有两层含义
- 创建一个新的Bean
- 从缓存中获取到已经被创建的对象

第一次调用肯定是创建一个新的Bean

首先调用getSingleton(String beanName)方法，这个方法又会调用返回getSingleton(beanName, true)

getSingleton(beanName, true)这个方法实际上就是到缓存中尝试去获取Bean，整个缓存分为三级

- singletonObjects，一级缓存，存储的是所有创建好了的单例Bean
- earlySingletonObjects，完成实例化，但是还未进行属性注入及初始化的对象
- singletonFactories，提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

因为A是第一次被创建，所以不管哪个缓存中必然都是没有的，因此会进入getSingleton的另外一个重载方法getSingleton(beanName, singletonFactory)

这个方法就是用来创建Bean的， 通过createBean方法返回的Bean最终被放到了一级缓存，也就是单例池中。

结论：一级缓存中存储的是已经完全创建好了的单例Bean

调用addSingletonFactory()方法<br/>
在完成Bean的实例化后，属性注入之前，Spring将Bean包装成一个工厂添加进了三级缓存中

这里只是添加了一个工厂，通过这个工厂（ObjectFactory）的getObject方法可以得到一个对象，而这个对象实际上就是通过getEarlyBeanReference这个方法创建的

当A完成了实例化并添加进了三级缓存后，就要开始为A进行属性注入了，在注入时发现A依赖了B，那么这个时候Spring又会去getBean(b)，然后反射调用setter方法完成属性注入

B需要注入A，所以在创建B的时候，又会去调用getBean(a)，这个时候就又回到之前的流程了，但是不同的是，之前的getBean是为了创建Bean，而此时再调用getBean不是为了创建了，而是要从缓存中获取，因为之前A在实例化后已经将其放入了三级缓存singletonFactories中，所以此时getBean(a)的流程就是这样子了

![004](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/004.png)

注入到B中的A是通过getEarlyBeanReference方法提前暴露出去的一个对象

当没有开启AOP的情况时，这个工厂啥都没干，getEarlyBeanReference其实原封不动的直接将实例化阶段创建的a对象返回了

### 完整流程

![005](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/005.png)

从上图中我们可以看到，虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以这是没有问题的

### 结合了AOP的循环依赖

如果在开启AOP的情况下，getEarlyBeanReference方法会调用到AnnotationAwareAspectJAutoProxyCreator的getEarlyBeanReference方法

将返回一个代理后的对象，而不是实例化阶段创建的对象，这样就意味着B中注入的A将是一个代理对象而不是A的实例化阶段创建后的对象

![006](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/006.png)

![007](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/QuestionBank/Spring%E7%9A%84%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96/007.png)

### Q&A
- 在给B注入的时候为什么要注入一个代理对象？

答：当我们对A进行了AOP代理时，说明我们希望从容器中获取到的就是A代理后的对象而不是A本身，因此把A当作依赖进行注入时也要注入它的代理对象

- 初始化的时候是对A对象本身进行初始化，而容器中以及注入到B中的都是代理对象，这样不会有问题吗？

答：不会，这是因为不管是cglib代理还是jdk动态代理生成的代理类，内部都持有一个目标类的引用，当调用代理对象的方法时，实际会去调用目标对象的方法，A完成初始化相当于代理对象自身也完成了初始化

- 三级缓存为什么要使用工厂而不是直接使用引用？换而言之，为什么需要这个三级缓存，直接通过二级缓存暴露一个引用不行吗？

答：这个工厂的目的在于延迟对实例化阶段生成的对象的代理，只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象


## 总结

面试官：”Spring是如何解决的循环依赖？“

答：<br/>
Spring通过三级缓存解决了循环依赖，
- 其中一级缓存为单例池（singletonObjects）
- 二级缓存为早期曝光对象earlySingletonObjects
- 三级缓存为早期曝光对象工厂（singletonFactories）
  
当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，<br/>
如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，<br/>
如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。<br/>
当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，<br/>
此时的getBean(a)会从缓存中获取，<br/>
第一步，先获取到三级缓存中的工厂；<br/>
第二步，调用对象工工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。
<br/>紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。至此，循环依赖结束！

面试官：”为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？“

答：如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。

## 笔记
```java
spring的三级缓存：
L1：Map<String, Object> singletonObjects
L2: Map<String, Object> earlySingletonObjects
L3: Map<String, ObjectFactory<?>> singletonFactories

AbstractAutowireCapableBeanFactory
doGetBean getSingleton(beanName)->getSingleton(beanName, true)
getSingleton(String beanName, ObjectFactory<?> singletonFactory)
singletonFactory:createBean(beanName, mbd, args);

createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
addSingletonFactory方法：
this.singletonFactories.put(beanName, singletonFactory);
this.earlySingletonObjects.remove(beanName);
this.registeredSingletons.add(beanName);

getEarlyBeanReference: 处理AOP ->
遍寻SmartInstantiationAwareBeanPostProcessor（AnnotationAwareAspectJAutoProxyCreator）
生存代理类
继续：
populateBean
initializeBean： 里面也会二次aop的后置处理器
AnnotationAwareAs
```

## 思考
依赖情况|依赖注入方式|循环依赖是否被解决
---|---|---
AB相互依赖（循环依赖）|均采用setter方法注入|是
AB相互依赖（循环依赖）|均采用构造器注入|否
AB相互依赖（循环依赖）|A中注入B的方式为setter方法，B中注入A的方式为构造器|是
AB相互依赖（循环依赖）|A中注入B的方式为构造器，B中注入A的方式为setter方法|否

为什么在上表中的第三种情况的循环依赖能被解决，而第四种情况不能被解决呢？

答

A、B两个实例循环依赖，因为 spring 默认按照字母排序创建 bean，如果 A 以构造器方式注入 B，那么 B 不论以什么方式注入 A，都会导致循环依赖进入死循环，因为整个过程中 A 实例从来没有出现过，就没有办法打破循环



