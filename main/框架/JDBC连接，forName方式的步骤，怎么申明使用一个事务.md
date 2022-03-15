# JDBC连接，forName方式的步骤，怎么声明一个事务

答：**jdbc连接步骤：**
- 注册驱动
- 获得连接
- 得到执行sql语句的statement
- 获得结果集
- 关闭资源

JDBC操作数据库的基本步骤：
- 1、**加载注册JDBC驱动程序**</br>
  在连接数据库之前，首先要加载想要连接的数据库的驱动到JVM（Java虚拟机）， 这通过java.lang.Class类的静态方法forName(String className)实现。成功加载后，会将Driver类的实例注册到DriverManager类中。
- 2、**提供JDBC连接的URL**</br>
  连接URL定义了连接数据库时的协议、子协议、数据源标识。`书写形式：协议：子协议：数据源标识`
>例如：jdbc:mysql://127.0.0.1:3306/jtdb?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&allowMultiQueries=true
- 3、**获取数据库的连接**</br>
  要连接数据库，需要向java.sql.DriverManager请求并获得Connection对象， 该对象就代表一个数据库的连接。</br>
  使用DriverManager的getConnectin(String url , String username , String password )方法传入指定的欲连接的数据库的路径、数据库的用户名和 密码来获得。
- 4、**得到执行sql语句的statement**</br>
  创建一个Statement，要执行SQL语句，必须获得java.sql.Statement实例。</br>
  Statement实例分为以下3 种类型：
    - （1）执行静态SQL语句。通常通过Statement实例实现。
    - （2）执行动态SQL语句。通常通过PreparedStatement实例实现。
    - （3）执行数据库存储过程。通常通过CallableStatement实例实现。
- 5、**执行SQL语句**</br>
  Statement接口提供了三种执行SQL语句的方法：executeQuery 、executeUpdate 和execute
- 6、**获得结果集**</br>
  ResultSet包含符合SQL语句中条件的所有行，并且它通过一套get方法提供了对这些行中数据的访问。</br>
  使用结果集（ResultSet）对象的访问方法获取数据。
    - （1）执行更新返回的是本次操作影响到的记录数。
    - （2）执行查询返回的结果是一个ResultSet对象。
- 7、**关闭JDBC对象（关闭结果集-->关闭数据库操作对象-->关闭连接）**</br>
  操作完成以后要把所有使用的JDBC对象全都关闭，以释放JDBC资源，`关闭顺序和声明顺序相反`：
    - （1）关闭记录集。
    - （2）关闭声明。
    - （3）关闭连接对象。

不建议使用class.fornamed的方式注册驱动，一般用DriverManager这个类的静态方法。</br>

**怎么申明使用一个事务?**</br>
JDBC中事务默认自动提交，可以用调用`setAutoCommit(false)`来禁止自动提交。在spring中，可以直接在配置文件中配置，然后使用注解就可以了。

Connection提供了事务处理的方法，通过调用setAutoCommit(false)可以设置手动提交事务；

当事务完成后用commit()显式提交事务；如果在事务处理过程中发生异常则通过rollback()进行事务回滚