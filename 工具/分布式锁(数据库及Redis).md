# 分布式锁（数据库和Redis）

由于多用户`多线程`条件下`同时(并发)` 对  `同一资源`进行访问，造成了线程并发问题。

一定时间`只允许一个线程`访问`共享资源`。当线程结束之后，才允许其他线程访问。

**Java中常见的锁用法(同步锁)**

```
graph LR
同步锁-->Synchronized
同步锁-->Lock
Synchronized-->每次只允许一个线程访问
Lock-->每次只允许一个线程访问
```
Lock是一种接口，一般通过其实现类ReenTrantLock来创建锁对象，然后通过lock()和unlock()规定加锁的代码区域。

![mskt_53](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_53.png)

`同步锁锁的是当前的线程`

阻塞严重，而且只有一个服务器，还是很容易造成瘫痪。

![mskt_50](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_50.png)

**什么是分布式？**

使用多台服务器搭建项目抗击高并发

总结：
- 多台服务器运行的都是一套完整的代码，这就叫集群。
- 多台服务器联合起来运行的才是一套完整代码，这就叫分布式。
- 在分布式这种横向拆分的基础上又做了纵向拆分。就变成SOA架构(面向服务)。

**分布式环境下同步锁的问题？**

如果使用分布式下的同步锁会发生超卖，资源会被分布式服务器分别操作。

Java的同步锁机制无法在分布式下对临界资源进行控制

![mskt_51](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_51.png)

**分布式锁**

概念：分布式锁是控制`分布式系统`之间`共同访问共享资源`的一种方式

**Mysql的分布式锁设计**

设计方案:
- 锁设计: 数据库中的表有主键，主键不能重复，如果插入一个重复的主键，则数据库必然报错。
- 数据库加锁: 向主键添加一个固定值 1；
- 数据库解锁: 从表中删除主键固定值 1；

代码实现

```java
@Mapper
public interface MysqlMapper {
	@Insert("insert into mysqllocl(id) values(1)")
	void insert();
	@Delete("delete from mysqllock where id = 1")
	void delete();
}
```
```java
@Service("mysqlLock")
public class MysqlLock implements Lock{
	@Autowired
	private MysqlMapper mapper;
	
	//表示尝试加锁
	@Override
	public boolean tryLock() {
		try {
			mapper.insert();//向数据库插入记录，固定值
		}catch (Exception e) {
			return false;//加锁失败
		}
		return true;//枷锁成功
	}
	//表示加锁机制
	@Override
	public void lock() {
		if(tryLock()) {
			//表示加锁成功
			return;
		}
		try {
			Thread.sleep(100);
		}catch (InterruptedException e) {
			e.printStackTrace();
		}
		lock();//递归加锁，加不上锁就不断尝试
	}
	//解锁操作
	@Override
	public void unlock() {
		mapper.delete();
	}
	
	//其他尚未实现
	@Override
	public void lockInterruptibly() throws InterruptedException {
		// TODO Auto-generated method stub
		
	}
	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		// TODO Auto-generated method stub
		return false;
	}
	@Override
	public Condition newCondition() {
		// TODO Auto-generated method stub
		return null;
	}
}
```
缺陷：
- 数据库锁性能低
- 可能造成死锁问题，比如一个线程在执行代码过程中突然挂掉。
- Mysql的加锁/解锁操作不是原子性操作。原子性(要么全部成功，要么全部失败)

**Redis实现分布式锁**

原则问题
- 互斥性：保证同一时间只有一个客户端可以获取锁(你加了锁别人不能再加)
- 避免死锁
- 原子性：保证枷锁解锁操作都是原子的(要么全部成功，要么全部失败)
- 安全性：只有加锁的那个服务才有解锁的权限。

![mskt_54](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_54.png)

Redis介绍
- Redis是高性能的KEY-VALUE结构存储系统
- Redis可以当作数据库/高速缓存/消息中间件使用
- Redis并发能力强，可以实现10万次/秒的操作
- Redis是单进程/单线程的操作，采用队列模式将并发访问变成串行访问
- 提供原子性的操作方式

![mskt_52](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_52.png)

`这是一种乐观锁，没有对数据库进行加锁，只是对第三方进行加锁，数据库不用做任何修改。`

**Redis互斥性实现**

存在问题:
```
redis.set(k,v)
```
任何线程都可以向Redis中添加数据，并且操作相同的KEY会覆盖原有数据。

解决方案:
```
long result = redis.setnx(key,value)
```
如果key不存在，可以赋值 返回值1</br>
如果key存在，不能复制 返回值0

**Redis避免死锁**

问题：</br>
如果加锁操作后，执行业务代码时出现异常，则不能执行解锁操作，会出现死锁现象。

![mskt_55](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_55.png)

解决方案
```
setex(key,seconds,value);
```
为数据添加超时时间，超时时间一到，则数据自动删除。

**Redis实现新增数据原子性**

如何保证`redis.setnx(key,value)防止重复操作`和`redis.setex(key.seconds,value)防止死锁`的操作的原子性？

解决方案：
- 命令：SET KEY VALUE NX PX 毫秒数
- NX: 防止重复操作，保证了互斥性
- PX: 设定超时时间，单位毫秒，防止死锁。
```
redis.set("LOCK","DATA","NX","PX",500);
```

**Redis保证数据安全性**

如何保证A线程加的锁，不被B线程提前释放呢？

解决思路
- 当加锁赋值时，设定随机数UUID
- 当解锁操作del时，对比UUID是否相同。相同则成功解锁。

![mskt_56](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_56.png)

**Redis实现解锁原子性**

问题：</br>
当A线程超时，这时B线程操作资源进行加锁，两个线程同时操作同一资源，A线程执行解锁操作，却将B线程的锁提前释放了

![mskt_57](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_57.png)

解决办法
- 采用LUA脚本保证解锁操作的原子性
```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```
取值，比较，删除等操作必须一气呵成，要么同时完成，要么同时失败，这就是原子性操作。

`总结：Redis锁的设计理念`
![mskt_59](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_59.png)

代码实现:
```java
@Component("redisLock")
public class RedisLock implements Lock{
	@Autowired
	private JedisCluster jedisCluster;
	private ThreadLocal<String> thread = new ThreadLocal<>();
	private static final String key = "JT_LOCK";
	//表示尝试加锁
	@Override
	public boolean tryLock() {
		try {
			String value = UUID.randomUUID().toString();
			String result = jedisCluster.set(key, value, "NX", "PX", 800);
			if(result.equalsIgnoreCase("ok")) {
				thread.set(value);
				return true;	//加锁成功!!!
			}
		} catch (Exception e) {
			return false;		//加锁失败
		}	
		return false;			//加锁失败
	}
	//表示加锁机制
	@Override
	public void lock() {
		if(tryLock()) {
			//表示加锁成功
			return;
		}
		try {
			Thread.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		lock();
	}
	@Override
	public void unlock() {
		String	uuid = thread.get();
		//所以使用lua脚本实现.保证数据的原子性操作
		String script = LuaStream.getLuaFileString();
		jedisCluster.eval(script, Arrays.asList(key), Arrays.asList(uuid));
	}
	
	//下面的尚未实现
	@Override
	public void lockInterruptibly() throws InterruptedException {
		// TODO Auto-generated method stub
	}
	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		// TODO Auto-generated method stub
		return false;
	}
	@Override
	public Condition newCondition() {
		// TODO Auto-generated method stub
		return null;
	}
}
```
```java
@Mapper
public interface MysqlMapper {
	@Insert("insert into mysqllock(id) values(1)")
	void insert();
	@Delete("delete from mysqllock where id = 1")
	void delete();
}
```
```java
public class LuaStream {
	@SuppressWarnings("resource")
	public static String getLuaFileString() {
		String script = null;
		try {
			InputStream fileInputStream = new FileInputStream("src/main/resources/lua/redis-lock.lua");
			//获取需要读取的字符串长度
			byte[] bytes = new byte[fileInputStream.available()];
			fileInputStream.read(bytes);
			script = new String(bytes);
			fileInputStream.close(); //关闭流
		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException();
		}
		return script;
	}
}
```