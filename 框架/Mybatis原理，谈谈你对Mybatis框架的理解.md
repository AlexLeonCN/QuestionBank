# Mybatis原理，谈谈你对Mybatis框架的理解

MyBatis是一个ORM持久层框架，可以用来实现数据库和内存之间的数据转化。起主要作用是封装了JDBC的操作，其实确切的来讲，MyBatis封装的是JDBC的API接口，也就是制定了一套规范，不同厂家基于这套规范有不同的实现，比如JDBC驱动有mysql, oracle,DB2等等。所以Mybatis对外提供接口，对内提供了接口的实现。

![mskt_24](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_24.png)

Mybatis通过io流读取配置文件(Mybatis-Config.xml)和映射文件(XxxMapper.xml)。通过XmlConfigBuilder读取配置文件中的元素，比如将环境信息封装成Environment对象，将数据源信息封装成DataSource对象等。通过XmlMapperBuilder读取映射文件中的元素，将Sql映射封装成一个个MappedStatement。最后它们被整体封装成Configuration配置对象，这里应用到简单工厂的设计模式。SqlSessionFactoryBuilder基于Configuration对象，通过build()方法创建SqlSessionFactory工厂对象，这里应用了建造模式的设计模式。SqlSessionFactory通过openSession()方法创建SqlSession，这里应用了工厂模式的设计模式。这部分就是Mybatis封装的API接口的创建过程。

![mskt_25](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_25.png)

而SqlSession对象中引用了Executor接口，Executor接口有多种实现：SimpleExecutor, CachingExecutor在SimpleExecutor的基础上扩展了缓存功能，BatchExecutor扩展了批量处理的功能。Executor会创建连接，并将连接，参数，MappedStatement交给StatementHandler接口，SetatementHandler接口也有多种实现，SimpleStatementHandler, CallebleStatementHandler, 以及PreparedStatementHandler预加载功能，可以防止Sql注入攻击。StatementHandler会基于MappedStatement创建Statement。再由ParameterHandler将语句中?位置填充参数；再由ResultSetHandler结果集处理器处理数据库端返回的结果。最终由TypeHandler处理数据库和Java之间的数据类型转换。

![mskt_26](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_26.png)
