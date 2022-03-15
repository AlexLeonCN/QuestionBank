# spring AOP的原理

AOP（Aspect Orient Programming）是一种设计思想，是软件设计领域中的面向切面编程，它是面向对象编程(OOP)的一种补充和完善。实际项目中我们通常将面向对象理解为一个静态过程(例如一个系统有多少个模块，一个模块有哪些对象，对象有哪些属性)，面向切面理解为一个动态过程（在对象运行时动态织入一些扩展功能或控制对象执行）

实际项目中通常会将系统分为两大部分，一部分是核心业务，一部分是非核业务。在编程实现时我们首先要完成的是核心业务的实现，非核心业务一般是通过特定方式切入到系统中，这种特定方式一般就是借助AOP进行实现。

AOP就是要基于OCP(开闭原则)，在不改变原有系统核心业务代码的基础上动态添加一些扩展功能并可以"控制"对象的执行。例如AOP应用于项目中的日志处理，事务处理，权限处理，缓存处理等等。

## AOP 应用原理分析

Spring AOP底层基于代理机制实现功能扩展：
- 假如目标对象(被代理对象)实现接口，则底层可以采用JDK动态代理机制为目标对象创建代理对象（目标类和代理类会实现共同接口）。
- 假如目标对象(被代理对象)没有实现接口，则底层可以采用CGLIB代理机制为目标对象创建代理对象（默认创建的代理类会继承目标对象类型）。

![mskt_31](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_31.png)

**基于JDK代理方式实现**</br>
当目标对象实现了接口，然后基于JDK代理方式进行功能织入，其原理如图所示

![mskt_33](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_33.png)

**基于CGLIB代理方式实现**</br>
假如目标对象没有实现接口，可以基于CGLIB代理方式为目标织入功能扩展，如图：

![mskt_34](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_34.png)

SpringBoot中AOP现在默认使用的CGLIB代理,假如需要使用JDK动态代理可以在配置文件(applicatiion.properties)中进行如下配置:
```yml
spring.aop.proxy-target-class=false
```
**AOP 相关术语分析**
- 切面(aspect): 横切面对象,一般为一个具体类对象(可以借助@Aspect声明)。
- 通知(Advice):在切面的某个特定连接点上执行的动作(扩展功能)，例如around,before,after等。
- 切入点(pointcut):对连接点拦截内容的一种定义,一般可以理解为多个连接点的结合。
- 连接点(joinpoint):程序执行过程中某个特定的点，一般指被拦截到的的方法。
  连接点与切入点定义如图所示：
  ![mskt_32](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_32.png)

**注解说明**
- @Aspect 注解用于标识或者说描述AOP中的切面类型，基于切面类型构建的对象用于为目标对象进行功能扩展或控制目标对象的执行。
- @Pointcut注解用于描述切面中的方法，并定义切面中的切入点（基于特定表达式的方式进行描述），在本案例中切入点表达式用的是bean表达式，这个表达式以bean开头，bean括号中的内容为一个spring管理的某个bean对象的名字。
- @Around注解用于描述切面中方法，这样的方法会被认为是一个环绕通知（核心业务方法执行之前和之后要执行的一个动作），@Aournd注解内部value属性的值为一个切入点表达式或者是切入点表达式的一个引用(这个引用为一个@PointCut注解描述的方法的方法名)。
- ProceedingJoinPoint类为一个连接点类型，此类型的对象用于封装要执行的目标方法相关的一些信息。一般用于@Around注解描述的方法参数。

**切面构造要素**</br>
`切面=切入点+通知`

在基于Spring AOP编程的过程中，基于AspectJ框架标准，spring中定义了五种类型的通知，它们分别是：
- 1)前置通知: ，目标方法执行之前执行
- 2)后置通知afterReturn，目标方法执行之后执行
- 3)环绕通知，使用最多并且功能强大，能够控制目标方法是否执行。
- 4)异常通知，当目标方法出现异常时执行
- 5)最终通知after，最后都要执行的通知.

事务/缓存/异常处理 可能会使用环绕通知，5大通知类型中环绕通知的功能最为强大，如果需要控制目标方法的执行使用环绕！！*

**通知执行顺序**</br>
假如这些通知全部写到一个切面对象中，其执行顺序及过程，如图所示：

![mskt_35](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_35.png)

**切入点表达式增强**</br>
Spring中通过切入点表达式定义具体切入点，其常用AOP切入点表达式定义及说明：
指示符 | 作用
---|---
bean | 用于匹配指定bean id的的方法执行
within | 用于匹配指定包名下类型内的方法执行
execution | 用于进行细粒度方法匹配执行具体业务
@annotation | 用于匹配指定注解修饰的方法执行

**bean表达式（重点）** `按照某个Bean(类)匹配拦截`</br>
**bean表达式一般应用于类级别，实现粗粒度的切入点定义，案例分析：**
- bean("userServiceImpl")指定一个userServiceImpl类中所有方法。
- bean("*ServiceImpl")指定所有的后缀为serviceImpl的类。

说明:bean表达式内部的对象是由spring容器管理的一个bean对象,表达式内部的内部的名字应该时spring容器中某个bean的key。

**within表达式（了解）** `按照类匹配拦截，可以加通配符，按照任意类，或同种类拦截 【粗粒度】`。</br>
**within表达式应用于类级别，实现粗粒度的切入点表达式定义，案例分析：**
- within("aop.service.UserServiceImpl")指定当前包中这个类内部的所有方法。
- within("aop.service.*") 指定当前目录下的所有类的所有方法。
- within("aop.service..*") 指定当前目录以及子目录中类的所有方法。

**execution表达式（了解）** `返回值类型 包名.类名.方法名(参数列表)) 【细粒度，方法参数级别】`</br>
**execution表达式应用于方法级别，实现细粒度的切入点表达式定义，案例分析**：

语法：execution(返回值类型 包名.类名.方法名(参数列表))。
- execution(void aop.service.UserServiceImpl.addUser())匹配addUser方法。
- execution(void aop.service.PersonServiceImpl.addUser(String)) 方法参数必须为String的addUser方法。
- execution(* aop.service..*.*(..)) 万能配置。
- excution(*com.jt..*.*(..))， jt包底下的所有类的所有方法，com.jt.*是指jt包下面的一级子包，com.jt..*只得是jt包下面所有子孙包，方法参数(..)是通配符指所有参数
- excution(* com.jt..*.add(..)), jt包底下所有包的add方法
- excutioon(* com.jt..*.a*(int)), jt包底下所有包的 a开头的方法，且参数只能有一个，并且参数必须是int类型

**@annotation表达式（重点）**</br>
@**annotaion表达式应用于方法级别，实现细粒度的切入点表达式定义，案例分析**
- @annotation(anno.RequiredLog) 匹配有此注解描述的方法。
- @annotation(anno.RequiredCache) 匹配有此注解描述的方法。

其中:RequestLog为我们自己定义的注解,当我们使用@RequiredLog注解修饰业务层方法时,系统底层会在执行此方法时进行扩展操作。

**切面优先级设置实现**</br>
切面的优先级需要借助@Order注解进行描述，数字越小优先级越高，默认优先级比较低。例如：</br>
定义日志切面并指定优先级。
```java
@Order(1)
@Aspect
@Component
public class SysLogAspect {
 …
}
```
定义缓存切面并指定优先级：
```java
@Order(2)
@Aspect
@Component
public class SysCacheAspect {
	…
}
```
说明：当多个切面作用于同一个目标对象方法时，这些切面会构建成一个切面链，类似过滤器链、拦截器链，其执行分析如图所示：
![mskt_36](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_36.png)