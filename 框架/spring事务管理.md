# spring事务管理

答: 所谓的事物管理指的是“按照指定事物规则执行提交或者回滚操作”，而spring的事物管理，指的是spring对事物管理规则的抽象。具体来说就是spring不直接管理事物，而是为了不同的平台(如jdbc hibernate jpa 等)提供了不同的事物管理器 ，将具体的事物管理委托给相应的平台实现。

不同的事务管理器都继承自`org.springframework.transaction.support.AbstractPlatformTransactionManager`抽象类，该抽象类实现了`org.springframework.jdbc.datasource.PlatformTransactionManager`接口。jdbc 事物管理器是通过javax.sql.DataSource.getConnection() 获取javax.sql.Connection,通过javax.sql.Connection来进行事物的提交和回滚。即其底层是通过Connection对象进行事务控制。

spring 中使用事物分为两种</br>
一种是编程式事物
- (1)PlatformTransactionManager
- (2)模版类TransactionTemplate（推荐使用）

一种声明式是事物
- (1)Spring的<tx:advice>定义事务通知与AOP相关配置实现
- (2)@Transactional注解方式实现

而一般我们又比较常用@Transactional来实现事物声明。@Transactional 常用以下属性propagation，tionisolation，readOnly，rollbackFor，rollbackForClassName，noRollbackFor，noRollbackForClassName

@Transactional`利用环绕通知TransactionInterceptor实现事务的开启及关闭`。
- 如果在接口、实现类或方法上都指定了@Transactional 注解，则优先级顺序为方法>实现类>接口；
- 建议只在实现类或实现类的方法上使用@Transactional，而不要在接口上使用，这是因为如果使用JDK代理机制（基于接口的代理）是没问题；而使用使用CGLIB代理（继承）机制时就会遇到问题，因为`其使用基于类的代理而不是接口，这是因为接口上的@Transactional注解是“不能继承的”`；


## 详解

### 1 数据库事物
数据库事物，指的是数据库执行的逻辑单元，这个逻辑单元遵从ACID原则，用这个原则来保证数据的一致性和完整性。反应到数据库上就是一组sql，要么全部执行成功，要么全部执行失败。例如下面sql：

>--uid001账户余额增加100块
update user_account set account_balance=account_balance+100 where user_id='uid001';</br>

>--uid002账户余额减少100块
update user_account set account_balance=account_balance-100 where user_id='uid002';

### 2 ACID

特性 | 英文 | 说明
---|---|---
原子性 | Atomicity | 事物的逻辑单元由一系列操作组成，这一系列操作要么全部成功，要么全部失败，回滚到未执行前一刻的状态
一致性 | Consistency | 	无论操作结果是成功还是失败，数据的业务状态是保持一致的。如uid001给uid002转账，无论成功还是失败，最终uid001和uid002的账户总和是一致的。
隔离性 | Isolation | 数据库同时会存在多个事物，多个事物之间是隔离的，一个事物的操作不会影响到另一个事物的操作结果。但也不是完全隔离，数据库层面定义了多个隔离级别，不通隔离级别隔离的效果是不一样的，隔离的级别越高，数据一致性越好，但是并发度越弱。四个隔离级别由小到大如下：`Read uncommitted<Read committed<Repeatable read<Serializable`
持久性 | Durability |	事物提交后，对数据的修改时永久的。及时系统发生故障，也能在不会丢失对数据的修改。一般通过执行前写入日志的方式保证持久性，即使系统崩溃，也能在重启的过程中，通过日志恢复崩溃时处于执行状态的事物。


### 3 数据库事务的隔离级别

名称 | 说明
---|---
ISOLATION_READ_UNCOMMITTED | 这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。 这种隔离级别会产生脏读，不可重复读和幻像读。
ISOLATION_READ_COMMITTED | 保证一个事务修改的数据提交后才能被另外 一个事务读取。另外一个事务不能读取该事务未提交的数据。（oracle默认）
ISOLATION_REPEATABLE_READ | 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。 它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。 （mysql默认）
ISOLATION_SERIALIZABLE | 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读


### 4.事务的类型

1 数据库分为本地事务跟全局事务：
- 本地事务：普通事务，独立一个数据库，能保证在该数据库上操作的ACID。
- 分布式事务：涉及两个或多个数据库源的事务，即跨越多台同类或异类数据库的事务（由每台数据库的本地事务组成的），分布式事务旨在保证这些本地事务的所有操作的ACID，使事务可以跨越多台数据库；

2 Java事务类型分为JDBC事务跟JTA事务
- JDBC事务：即为上面说的数据库事务中的本地事务，通过connection对象控制管理。
- JTA事务：JTA指Java事务API(Java Transaction API)，是Java EE数据库事务规范， JTA`只提供了事务管理接口`，由应用程序服务器厂商（如WebSphere Application Server）提供实现，JTA事务比JDBC更强大，`支持分布式事务`。

3 按是否通过编程分为声明式事务和编程式事务：
- 声明式事务：通过XML配置或者注解实现。
- 编程式事务：通过编程代码在业务逻辑时需要时自行实现，粒度更小。

### 5.Spring事务隔离级别

spring有五大隔离级别，其在TransactionDefinition接口中定义。看源码可知，其默isolation_default（底层数据库默认级别），其他四个隔离级别跟数据库隔离级别一致。

- **ISOLATION_DEFAULT**：用底层数据库的默认隔离级别，数据库管理员设置什么就是什么
- **ISOLATION_READ_UNCOMMITTED（未提交读）**：最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）
- **ISOLATION_READ_COMMITTED（提交读）**：一个事务提交后才能被其他事务读取到（该隔离级别禁止其他事务读取到未提交事务的数据、所以还是会造成幻读、不可重复读）、sql server默认级别
- **ISOLATION_REPEATABLE_READ（可重复读）**：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（该隔离基本可防止脏读，不可重复读（重点在修改），但会出现幻读（重点在增加与删除））（MySql默认级别，更改可通过set transaction isolation level 级别）
- **ISOLATION_SERIALIZABLE（序列化）**：代价最高最可靠的隔离级别（该隔离级别能防止脏读、不可重复读、幻读）
    - 丢失更新：两个事务同时更新一行数据，最后一个事务的更新会覆盖掉第一个事务的更新，从而导致第一个事务更新的数据丢失，这是由于没有加锁造成的；
    - 幻读：同样的事务操作过程中，不同时间段多次（不同事务）读取同一数据，读取到的内容不一致（一般是行数变多或变少）。
    - 脏读：一个事务读取到另外一个未提及事务的内容，即为脏读。
    - 不可重复读：同一事务中，多次读取内容不一致（一般行数不变，而内容变了）。
      幻读与不可重复读的区别：幻读的重点在于`插入与删除`，即第二次查询会发现比第一次查询数据变少或者变多了，以至于给人一种幻象一样，而不可重复读重点在于`修改`，即第二次查询会发现查询结果比第一次查询结果不一致，即第一次结果已经不可重现了。

数据库隔离级别越高，执行代价越高，并发执行能力越差，因此在实际项目开发使用时要综合考虑，为了考虑并发性能一般使用`提交读隔离级别`，它能避免丢失更新和脏读，尽管不可重复读和幻读不能避免，但可以在可能出现的场合使用`悲观锁或乐观锁来`解决这些问题。

### 6 传播行为

有七大传播行为，也是在TransactionDefinition接口中定义。

- **PROPAGATION_REQUIRED**：支持当前事务，如当前没有事务，则新建一个。
- **PROPAGATION_SUPPORTS**：支持当前事务，如当前没有事务，则已非事务性执行（源码中提示有个注意点，看不太明白，留待后面考究）。
- **PROPAGATION_MANDATORY**：支持当前事务，如当前没有事务，则抛出异常（强制一定要在一个已经存在的事务中执行，业务方法不可独自发起自己的事务）。
- **PROPAGATION_REQUIRES_NEW**：始终新建一个事务，如当前原来有事务，则把原事务挂起。
- **PROPAGATION_NOT_SUPPORTED**：不支持当前事务，始终已非事务性方式执行，如当前事务存在，挂起该事务。
- **PROPAGATION_NEVER**：不支持当前事务；如果当前事务存在，则引发异常。
- **PROPAGATION_NESTED**：如果当前事务存在，则在嵌套事务中执行，如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作（注意：当应用到JDBC时，只适用JDBC 3.0以上驱动）。

### 7.Spring事务支持

内置事务管理器都继承了抽象类`AbstractPlatformTransactionManager`，而`AbstractPlatformTransactionManager`又继承了接口`PlatformTransactionManager`

Spring框架支持事务管理的核心是事务管理器抽象，对于不同的数据访问框架通过实现策略接口`PlatformTransactionManager`，从而能支持多钟数据访问框架的事务管理。

`PlatformTransactionManager1接口定义如下
```java
public interface PlatformTransactionManager {
         TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;//返回一个已经激活的事务或创建一个新的事务（具体由TransactionDefinition参数定义的事务属性决定），返回的TransactionStatus对象代表了当前事务的状态，其中该方法抛出TransactionException（未检查异常）表示事务由于某种原因失败。
         void commit(TransactionStatus status) throws TransactionException;//用于提交TransactionStatus参数代表的事务。
         void rollback(TransactionStatus status) throws TransactionException;//用于回滚TransactionStatus参数代表的事务。
｝
```
TransactionDefinition接口定义如下：
```java
public interface TransactionDefinition {  
       int getPropagationBehavior();  //返回定义的事务传播行为
       int getIsolationLevel(); //返回事务隔离级别
       int getTimeout();  //返回定义的事务超时时间
       boolean isReadOnly();  //返回定义的事务是否是只读的
       String getName();  //返回事务名称
}  
```
TransactionStatus接口定义如下：
```java
public interface TransactionStatus extends SavepointManager {  
       boolean isNewTransaction();  //返回当前事务是否是新的事务
       boolean hasSavepoint();  //返回当前事务是否有保存点
       void setRollbackOnly();  //设置事务回滚
       boolean isRollbackOnly();  //设置当前事务是否应该回滚
       void flush();  //用于刷新底层会话中的修改到数据库，一般用于刷新如Hibernate/JPA的会话，可能对如JDBC类型的事务无任何影响；
       boolean isCompleted();  //返回事务是否完成
}  
```
**Spring分布式事务配置(略)**</br>
https://blog.csdn.net/liaohaojian/article/details/68488150

### 8 Spring事务管理实现方式之编程式事务与声明式事务详解

**编程式事务：编码方式实现事务管理（代码演示为JDBC事务管理）**</br>
Spring实现编程式事务，依赖于2大类，PlatformTransactionManager，与模版类TransactionTemplate（推荐使用）。下面分别详细介绍Spring是如何通过该类实现事务管理。

1）PlatformTransactionManager</br>
事务管理器配置
```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="jdbcUrl" value="${db.jdbcUrl}" />
	<property name="user" value="${user}" />
	<property name="password" value="${password}" />
	<property name="driverClass" value="${db.driverClass}" />
	 <!--连接池中保留的最小连接数。 --> 
     <property name="minPoolSize"> 
         <value>5</value> 
     </property> 
     <!--连接池中保留的最大连接数。Default: 15 --> 
     <property name="maxPoolSize"> 
         <value>30</value> 
     </property> 
     <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 --> 
     <property name="initialPoolSize"> 
         <value>10</value> 
     </property> 
     <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 --> 
     <property name="maxIdleTime"> 
         <value>60</value> 
     </property> 
     <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 --> 
     <property name="acquireIncrement"> 
         <value>5</value> 
     </property> 
     <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements 属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。  如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0 --> 
     <property name="maxStatements"> 
         <value>0</value> 
     </property> 
     <!--每60秒检查所有连接池中的空闲连接。Default: 0 --> 
     <property name="idleConnectionTestPeriod"> 
         <value>60</value> 
     </property> 
     <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 --> 
     <property name="acquireRetryAttempts"> 
         <value>30</value> 
     </property> 
     <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效 保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。Default: false --> 
     <property name="breakAfterAcquireFailure"> 
         <value>true</value> 
     </property> 
     <!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的 时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable等方法来提升连接测试的性能。Default: false --> 
     <property name="testConnectionOnCheckout"> 
         <value>false</value> 
     </property> 
</bean>
<!--DataSourceTransactionManager位于org.springframework.jdbc.datasource包下，数据源事务管理类，提供对单个javax.sql.DataSource数据源的事务管理，主要用于JDBC，Mybatis框架事务管理。 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
```
业务中使用代码(以测试类展示)
```java
import java.util.Map;
import javax.annotation.Resource;
import javax.sql.DataSource;
import org.apache.log4j.Logger;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:spring-public.xml" })
public class test {
	@Resource
	private PlatformTransactionManager txManager;
	@Resource
	private  DataSource dataSource;
	private static JdbcTemplate jdbcTemplate;
	Logger logger=Logger.getLogger(test.class);
    private static final String INSERT_SQL = "insert into testtranstation(sd) values(?)";
    private static final String COUNT_SQL = "select count(*) from testtranstation";
	@Test
	public void testdelivery(){
		//定义事务隔离级别，传播行为，
	    DefaultTransactionDefinition def = new DefaultTransactionDefinition();  
	    def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);  
	    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);  
	    //事务状态类，通过PlatformTransactionManager的getTransaction方法根据事务定义获取；获取事务状态后，Spring根据传播行为来决定如何开启事务
	    TransactionStatus status = txManager.getTransaction(def);  
	    jdbcTemplate = new JdbcTemplate(dataSource);
	    int i = jdbcTemplate.queryForInt(COUNT_SQL);  
	    System.out.println("表中记录总数："+i);
	    try {  
	        jdbcTemplate.update(INSERT_SQL, "1");  
	        txManager.commit(status);  //提交status中绑定的事务
	    } catch (RuntimeException e) {  
	        txManager.rollback(status);  //回滚
	    }  
	    i = jdbcTemplate.queryForInt(COUNT_SQL);  
	    System.out.println("表中记录总数："+i);
	}
	
}
```
2）使用TransactionTemplate，该类继承了接口`DefaultTransactionDefinitio`n，用于简化事务管理，事务管理由模板类定义，主要是通过TransactionCallback回调接口或TransactionCallbackWithoutResult回调接口指定，通过调用模板类的参数类型为TransactionCallback或TransactionCallbackWithoutResult的execute方法来自动享受事务管理。

TransactionTemplate模板类使用的回调接口：
- TransactionCallback：通过实现该接口的“T doInTransaction(TransactionStatus status) ”方法来定义需要事务管理的操作代码；
- TransactionCallbackWithoutResult：继承TransactionCallback接口，提供“void doInTransactionWithoutResult(TransactionStatus status)”便利接口用于方便那些不需要返回值的事务操作代码。

还是以测试类方式展示如何实现
```java
@Test
public void testTransactionTemplate(){
	jdbcTemplate = new JdbcTemplate(dataSource);
    int i = jdbcTemplate.queryForInt(COUNT_SQL);  
    System.out.println("表中记录总数："+i);
	//构造函数初始化TransactionTemplate
	TransactionTemplate template = new TransactionTemplate(txManager);
	template.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);  
	//重写execute方法实现事务管理
	template.execute(new TransactionCallbackWithoutResult() {
		@Override
		protected void doInTransactionWithoutResult(TransactionStatus status) {
			jdbcTemplate.update(INSERT_SQL, "饿死");   //字段sd为int型，所以插入肯定失败报异常，自动回滚，代表TransactionTemplate自动管理事务
		}}
	);
	i = jdbcTemplate.queryForInt(COUNT_SQL);  
    System.out.println("表中记录总数："+i);
}
```
**声明式事务**：可知编程式事务每次实现都要单独实现，但业务量大功能复杂时，使用编程式事务无疑是痛苦的，而声明式事务不同，声明式事务属于无侵入式，不会影响业务逻辑的实现。

声明式事务实现方式主要有2种，一种为通过使用Spring的<tx:advice>定义事务通知与AOP相关配置实现，另为一种通过@Transactional实现事务管理实现，下面详细说明2种方法如何配置，已经相关注意点

1)方式一，配置文件如下
```xml
<!-- 
<tx:advice>定义事务通知，用于指定事务属性，其中“transaction-manager”属性指定事务管理器，并通过<tx:attributes>指定具体需要拦截的方法
	<tx:method>拦截方法，其中参数有：
	name:方法名称，将匹配的方法注入事务管理，可用通配符
	propagation：事务传播行为，
	isolation：事务隔离级别定义；默认为“DEFAULT”
	timeout：事务超时时间设置，单位为秒，默认-1，表示事务超时将依赖于底层事务系统；
	read-only：事务只读设置，默认为false，表示不是只读；
    rollback-for：需要触发回滚的异常定义，可定义多个，以“，”分割，默认任何RuntimeException都将导致事务回滚，而任何Checked Exception将不导致事务回滚；
    no-rollback-for：不被触发进行回滚的 Exception(s)；可定义多个，以“，”分割；
 -->
<tx:advice id="advice" transaction-manager="transactionManager">
	<tx:attributes>
	    <!-- 拦截save开头的方法，事务传播行为为：REQUIRED：必须要有事务, 如果没有就在上下文创建一个 -->
		<tx:method name="save*" propagation="REQUIRED" isolation="READ_COMMITTED" timeout="" read-only="false" no-rollback-for="" rollback-for=""/>
		<!-- 支持,如果有就有,没有就没有 -->
		<tx:method name="*" propagation="SUPPORTS"/>
	</tx:attributes>
</tx:advice>
<!-- 定义切入点，expression为切人点表达式，如下是指定impl包下的所有方法，具体以自身实际要求自定义  -->
<aop:config>
    <aop:pointcut expression="execution(* com.kaizhi.*.service.impl.*.*(..))" id="pointcut"/>
    <!--<aop:advisor>定义切入点，与通知，把tx与aop的配置关联,才是完整的声明事务配置 -->
    <aop:advisor advice-ref="advice" pointcut-ref="pointcut"/>
</aop:config>
```
注意点：事务回滚异常只能为RuntimeException异常，而Checked Exception异常不回滚，捕获异常不抛出也不会回滚，但可以强制事务回滚：`TransactionAspectSupport.currentTransactionStatus().isRollbackOnly();`

2）方式二通过@Transactional实现事务管理
```java
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">   
     <property name="dataSource" ref="dataSource"/>
</bean>    
<tx:annotation-driven transaction-manager="txManager"/> //开启事务注解
```
`@Transactional(propagation=Propagation.REQUIRED,isolation=Isolation.READ_COMMITTED)`，具体参数跟上面<tx:method>中一样
`Spring提供的@Transaction注解事务管理，内部同样是利用环绕通知TransactionInterceptor实现事务的开启及关闭。`

使用@Transactional注意点：
- 如果在接口、实现类或方法上都指定了@Transactional 注解，则优先级顺序为`方法>实现类>接口`；
- `建议只在实现类或实现类的方法上使用@Transactional，而不要在接口上使用，这是因为如果使用JDK代理机制（基于接口的代理）是没问题；而使用使用CGLIB代理（继承）机制时就会遇到问题，因为其使用基于类的代理而不是接口，这是因为接口上的@Transactional注解是“不能继承的”；`