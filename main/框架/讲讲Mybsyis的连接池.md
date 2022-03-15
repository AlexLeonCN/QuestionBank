# 讲讲Mybsyis的连接池

连接池有很多种，最为熟悉的比如c3p0，DBCP，druid等。

mybatis支持三种内置的数据源类型：
- Pooled：实现dataSource接口，并且使用了池的思想。
- UNPooled：同样也是实现了dataSource接口，但是该类型并没有使用池的思想。
- JDNI：采用服务器提供的JDNI技术实现的，并且在不同服务器之间获取的连接池是不一样的。

**注意**：如果项目不是web或者maven的war工程，则是无法使用的。比如Tomcat服务器采用的就是DBCP连接池。

那么在Mybatis种如何配置数据源，如下所示：
```xml
<!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
            </dataSource>
```
在dataSource属性中，type标签可以指定所使用的数据源类型。

## 一、UnPooled
大致流程：
- 在获取连接对象时，调用`initializeDriver()`方法判断是否已经注册连接驱动。
- 完成驱动注册，使用`DriverManager.getConnection`获取一个连接对象。
- 将连接对象交给`configureConnection()`方法，并设置自动提交事务，及事务的隔离级别。
>这种方式是不具备连接池的思想，如果频繁的创建和销毁连接对象，会影响程序的运行效率。

1，首先从获取连接的源码开始。
```java
@Override
  public Connection getConnection() throws SQLException {
    return doGetConnection(username, password);
  }
```
2，接着再进入到`doGetConnection()`方法中：
```java
private Connection doGetConnection(String username, String password) throws SQLException {
    // 实例化一个集合
    Properties props = new Properties();
    if (driverProperties != null) {
      props.putAll(driverProperties);
    }
    // 判断用户名是否为空
    if (username != null) {
      props.setProperty("user", username);
    }
    // 判断密码是否为空
    if (password != null) {
      props.setProperty("password", password);
    }
    return doGetConnection(props);
  }
```
3，此时又返回一个`doGetConnection()`，这是重载的另一个方法。
```java
private Connection doGetConnection(Properties properties) throws SQLException {
    // 初始化
    initializeDriver();
    // 获取一个连接对象
    Connection connection = DriverManager.getConnection(url, properties);
    configureConnection(connection);
    return connection;
  }
```
4，第一行代码中，调用了`initializeDriver()`方法。
```java
private synchronized void initializeDriver() throws SQLException {
    if (!registeredDrivers.containsKey(driver)) {
      Class<?> driverType;
      try {
        if (driverClassLoader != null) {
            // 使用反射获取到连接驱动
          driverType = Class.forName(driver, true, driverClassLoader);
        } else {
          driverType = Resources.classForName(driver);
        }
        // DriverManager requires the driver to be loaded via the system ClassLoader.
        // http://www.kfu.com/~nsayer/Java/dyn-jdbc.html
        // 实例化连接驱动
          Driver driverInstance = (Driver)driverType.newInstance();
          // 注册驱动
        DriverManager.registerDriver(new DriverProxy(driverInstance));
        registeredDrivers.put(driver, driverInstance);
      } catch (Exception e) {
        throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + e);
      }
    }
  }
```
## 二，Pooled
再来看看使用连接池思想的数据源实现。</br>
在此之前先来说说什么是连接池，连接池就是用于存储连接对象的一个容器。而容器就是一个集合，且必须是线程安全的，即两个线程不能拿到同一个连接对象。同时还要具备队列的特性：先进先出原则。</br>
**使用连接池的好处**：避免频繁创建和关闭数据库连接造成的开销，节省系统资源。

分析：
- 1，判断连接池中是否有空闲的连接对象，有则直接返回。
- 2，如果连接池没有空闲的连接，先判断活动连接池是否小于连接池承载的最大数量，小于则再创建新的连接对象。
- 3，但是如果连接池已经达到最大承载数量，那么在连接池中就把最先进来的连接（oldest）返回出去。

1，先从获取连接源码开始。
```java
@Override
  public Connection getConnection() throws SQLException {
    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
  }
```
2，调用popConnection()方法（有点长)
```java
private PooledConnection popConnection(String username, String password) throws SQLException {
    boolean countedWait = false;
    PooledConnection conn = null;
    long t = System.currentTimeMillis();
    int localBadConnectionCount = 0;

    while (conn == null) {
      synchronized (state) {
        if (!state.idleConnections.isEmpty()) {
          // Pool has available connection
          conn = state.idleConnections.remove(0);
          if (log.isDebugEnabled()) {
            log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
          }
        } else {
          // Pool does not have available connection
          if (state.activeConnections.size() < poolMaximumActiveConnections) {
            // Can create new connection
            conn = new PooledConnection(dataSource.getConnection(), this);
            if (log.isDebugEnabled()) {
              log.debug("Created connection " + conn.getRealHashCode() + ".");
            }
          } else {
            // Cannot create new connection
            PooledConnection oldestActiveConnection = state.activeConnections.get(0);
            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
            if (longestCheckoutTime > poolMaximumCheckoutTime) {
              // Can claim overdue connection
              state.claimedOverdueConnectionCount++;
              state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
              state.accumulatedCheckoutTime += longestCheckoutTime;
              state.activeConnections.remove(oldestActiveConnection);
              if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                try {
                  oldestActiveConnection.getRealConnection().rollback();
                } catch (SQLException e) {
                  log.debug("Bad connection. Could not roll back");
                }  
              }
              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
              conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
              conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
              oldestActiveConnection.invalidate();
              if (log.isDebugEnabled()) {
                log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
              }
            } else {
              // Must wait
              try {
                if (!countedWait) {
                  state.hadToWaitCount++;
                  countedWait = true;
                }
                if (log.isDebugEnabled()) {
                  log.debug("Waiting as long as " + poolTimeToWait + " milliseconds for connection.");
                }
                long wt = System.currentTimeMillis();
                state.wait(poolTimeToWait);
                state.accumulatedWaitTime += System.currentTimeMillis() - wt;
              } catch (InterruptedException e) {
                break;
              }
            }
          }
        }
        if (conn != null) {
          // ping to server and check the connection is valid or not
          if (conn.isValid()) {
            if (!conn.getRealConnection().getAutoCommit()) {
              conn.getRealConnection().rollback();
            }
            conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));
            conn.setCheckoutTimestamp(System.currentTimeMillis());
            conn.setLastUsedTimestamp(System.currentTimeMillis());
            state.activeConnections.add(conn);
            state.requestCount++;
            state.accumulatedRequestTime += System.currentTimeMillis() - t;
          } else {
            if (log.isDebugEnabled()) {
              log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
            }
            state.badConnectionCount++;
            localBadConnectionCount++;
            conn = null;
            if (localBadConnectionCount > (poolMaximumIdleConnections + poolMaximumLocalBadConnectionTolerance)) {
              if (log.isDebugEnabled()) {
                log.debug("PooledDataSource: Could not get a good connection to the database.");
              }
              throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
            }
          }
        }
      }

    }

    if (conn == null) {
      if (log.isDebugEnabled()) {
        log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
      }
      throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
    }

    return conn;
  }
```