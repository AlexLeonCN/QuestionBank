# spring注入的几种方式(循环注入)

## 概述：

Spring支持四种依赖注入方式，分别是</br>
### 1 属性注入(setter):
使用setter方法注入，spring配置中采用property
```xml
<property name="brand" value="保时捷 k3"/>
```
### 2 构造函数注入
采用构造方法注入
- 1.按参数类型匹配入参: `constructor-arg type=`
    ```xml
    <bean id="car1" class="com.vtstar.entity.Car">
        <constructor-arg type="int" value="300"/>
        <constructor-arg type="java.lang.String" value="红旗"/>
        <constructor-arg type="double" value="20000000.9"/>
    </bean>
    ```
- 2.按参数索引匹配入参: `constructor-arg index=`
    ```xml
    <bean id="car2" class="com.vtstar.entity.Car">
        <constructor-arg index="0" value="400"/>
        <constructor-arg index="1" value="大众辉腾"/>
        <constructor-arg index="2" value="20000000"/>
    </bean>
    ```
- 3.联合使用参数类型和参数索引匹配入参
    ```xml
    <bean id="car3" class="com.vtstar.entity.Car">
        <constructor-arg index="0" type="java.lang.String" value="30000000.0"/>
        <constructor-arg index="1" type="java.lang.String" value="卡迪拉克"/>
        <constructor-arg index="2" type="int" value="400"/>
    </bean>
    ```
- 4.通过自身类型反射入参: 每个属性都是引用数据类型且类型都不同
    ```java
        <!--4.通过自身类型反射入参-->
    <bean id="boss" class="com.vtstar.entity.Boss">
        <constructor-arg value="Tom"/>
        <constructor-arg ref="car1"/>
        <constructor-arg value="20"/>
    </bean>
    ```
- 循环依赖问题:</br>由于Spring容器对构造函数配置Bean实例化要求所有Bean构造函数入参的对象都得是就绪的，所以如果两个Bean构造参数互相引用，就会导致线程死锁和循环依赖问题，解决办法：将相互依赖的两个Bean中的其中一个Bean采用Setter注入的方式即可。

### 3 工厂注入
首先需要创建工厂类，工厂类中使用 对象.set方法
```xml
    <!--非静态注入工厂方法-->
    <bean id="carFactory" class="com.vtstar.ioc.CarFactory"/>
    <bean id="car5" factory-bean="carFactory" factory-method="createHongQiCar"/>
    <!--静态注入工厂方法-->
    <bean id="car6" class="com.vtstar.ioc.CarFactory" factory-method="createDaZhongCar"/>
```
### 4 注解方式注入
- @Resource：java的注解，默认以byName的方式去匹配与属性名相同的bean的id，如果没有找到就会以byType的方式查找.
- @Autowired：spring注解，默认是以byType的方式去匹配与属性名相同的bean的id，如果没有找到，就通过byName的方式去查找，
- 如果byType查找到多个的话，还可以使用追加使用@Qualifier注解（spring注解）指定某个具体名称的bean。

## 详解：

### 一.属性注入
**定义**：属性注入是指通过 setXxx()方法注入Bean的属性或依赖对象。</br>
**优点**: 属性注入方式具有`可选择性和高灵活性`的特点，属性注入方式是实际应用中最常采用的注入方式。

Car类中定义了3个属性，并分别提供了对应的Setter方法
```java
package com.vtstar.entity;
 
/**
 * @ClassName Car
 * @Description TODO
 * @Author XiaoWu
 * @Date 2018/9/6 9:53
 * @Version 1.0
 **/
public class Car {
 
    private int maxSpeed;
 
    public String brand;
 
    private double price;
 
    private Boss boss;
 
    public Car() {
    }
 
    public Car(double price, Boss boss) {
        System.out.println("I'm the Car's construct two " + price + " boss " + boss.getName());
        this.price = price;
        this.boss = boss;
    }
 
    public Car(int maxSpeed, String brand, double price) {
        System.out.println("车的时速是" + maxSpeed + " 品牌为：" + brand + " 价格为：" + price);
        this.maxSpeed = maxSpeed;
        this.brand = brand;
        this.price = price;
    }
 
    public int getMaxSpeed() {
        return maxSpeed;
    }
 
    public void setMaxSpeed(int maxSpeed) {
        System.out.println("The maximum speed is " + maxSpeed);
        this.maxSpeed = maxSpeed;
    }
 
    public String getBrand() {
        return brand;
    }
 
    public void setBrand(String brand) {
        System.out.println("It is a " + brand + " Car");
        this.brand = brand;
    }
 
    public double getPrice() {
        return price;
    }
 
    public void setPrice(double price) {
        System.out.println("The price of this car " + price);
        this.price = price;
    }
}
```
在Spring配置文件中对Car进行属性注入的配置桥段
```xml
<!--&&&&&&&&&&&&&&&&&&&&&&&&& setter属性方式注入 &&&&&&&&&&&&&&&&&&&&&& -->
<bean id="car" class="com.vtstar.entity.Car">
     <property name="brand" value="保时捷 k3"/>
     <property name="maxSpeed" value="100"/>
     <property name="price" value="30.1"/>
</bean>
```
上面的代码配置的一个Bean,class属性对应前面建好的实体Car，<property/>标签对应Bean中的每一个属性,name为属性的名称，value为参数值，在Bean中拥有与其对应的Setter方法，maxSpeed对应setMaxSpeed(),price对应setPrice()；

### 二.构造函数注入

构造函数注入是除属性注入外的另外一种注入方式，它`保证了一些必要的属性在Bean实例化时就得到设置，确保在Bean实例化后就可以使用`。

值得一提的是构造函数注入又分为多种方式.

#### 1. 按类型匹配入参
如果任何可用的Car对象都需要使用到brand,maxSpeed,price的值，那使用Setter注入方式，则只能`人为的在配置时提供保证而无法再语法上提供保证`,那这个时候使用构造函数注入就能满足这一个要求，使用构造函数注入的前提是要保证`Bean中有提供带参的构造函数`。
```xml
     <!--1.根据参数类型注入-->
    <bean id="car1" class="com.vtstar.entity.Car">
        <constructor-arg type="int" value="300"/>
        <constructor-arg type="java.lang.String" value="红旗"/>
        <constructor-arg type="double" value="20000000.9"/>
    </bean>
```
在<constructor/>元素中有一个type元素，它为Spring提供了判断配置项和构造函数入参对应关系的“信息”。

#### 2. 按索引匹配入参
Java语言通过入参的类型及顺序区分不同的重载方法。`如果构造函数中有两个类型相同的入参`，那么使用第一种方式是行不通的，因为type无法确认对应的关系。这时我们就需要使用索引匹配入参的方式来进行确认。

为了更好的演示按索引匹配入参，将Car构造函数进行了修改
```java
 public Car(String brand, String corp, double price) {
        System.out.println("brand :" + brand + " corp :" + corp + " price :"+price);
        this.brand = brand;
        this.corp = corp;
        this.price = price;
 }
```
```xml
    <!--2.通过入参位置下标-->
    <bean id="car2" class="com.vtstar.entity.Car">
        <constructor-arg index="0" value="400"/>
        <constructor-arg index="1" value="大众辉腾"/>
        <constructor-arg index="2" value="20000000"/>
    </bean>
```
因为brand和corp都是String类型，所以Spring无法确定type为String的<constructor-arg/>到底对应的是brand还是corp。但是这种按索引匹配入参的方式能够`消除这种不确定性`。

#### 3. 联合使用类型和索引匹配入参
有时需要Type和index联合使用才能确定配置项和构造函数入参的对应关系，举个栗子
```java
    public Car(String brand, String corp, double price) {
        System.out.println("brand :" + brand + " corp :" + corp + " price :"+price);
        this.brand = brand;
        this.corp = corp;
        this.price = price;
    }
    public Car(String brand, String corp, int maxSpeed) {
        System.out.println("brand :" + brand + " corp :" + corp + " maxSpeed :"+maxSpeed);
        this.brand = brand;
        this.corp = corp;
        this.maxSpeed = maxSpeed;
    }
```
在这里，Car拥有两个重载的构造函数，它们都有两个相同类型的入参，按照index的方式针对这样的情况又难以满足了这时就需要联合使用<constructor-arg/>中的type和index了。
```xml
    <!--3.通过参数类型和入参位置联合注入-->
    <bean id="car3" class="com.vtstar.entity.Car">
        <constructor-arg index="0" type="java.lang.String" value="30000000.0"/>
        <constructor-arg index="1" type="java.lang.String" value="卡迪拉克"/>
        <constructor-arg index="2" type="int" value="400"/>
    </bean>
```
对于上图的代码清单如果只根据index来进行匹配入参，那么Spring无法确认第三个参数是price还是maxSpeed了，所以解决这种有歧义的冲突，请将type和index结合使用，对于`因参数数目相同而类型不同引起的潜在配置歧义问题`，Spring容器可以正确的启动且不会给出报错信息，他将`随机采用一个匹配的构造函数`实例化Bean,而被选择的构造函数可能并不是用户所期望的那个。因此，必须要特别谨慎，以避免潜在的错误。

#### 4.通过自身类型反射入参
如果Bean的构造函数入参类型是可辨别的，什么是可辨别的入参类型呢？（`非基础数据类型且入参类型各异`）

我们再建一个Boss实体类，在Boss类中引用Car类
```java
package com.vtstar.entity;
 
import java.util.Date;
 
/**
 * @ClassName Boss
 * @Description TODO
 * @Author XiaoWu
 * @Date 2018/9/6 11:22
 * @Version 1.0
 **/
public class Boss {
    private String name;
    private Car car;
    private Integer age;
 
    public Boss() {
    }
 
    public Boss(String name, Car car,Integer age) {
        System.out.println("The name of the boss " + name + " ,He has a  " + car.getBrand()+" age is "+age);
        this.name = name;
        this.car = car;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public Car getCar() {
        return car;
    }
 
    public Integer getAge() {
        return age;
    }
 
    public void setAge(Integer age) {
        this.age = age;
    }
}
```
Spring配置Boss相关的Bean，由于name, car ,age入参都是可辨别的，所以无须在<constructor-arg/>中指定type和index。
```xml
    <!--4.通过自身类型反射入参-->
    <bean id="boss" class="com.vtstar.entity.Boss">
        <constructor-arg value="Tom"/>
        <constructor-arg ref="car1"/>
        <constructor-arg value="20"/>
    </bean>
```
但是为了避免潜在配置歧义引起的张冠李戴的情况，如果Bean存在多个构造函数，那么`显式指定index和type属性是一种良好的配置习惯`。

`5.循环依赖问题`</br>
Spring容器对构造函数配置Bean进行实例化有一个前提，即`Bean构造函数入参引用的对象必须已经准备就绪`。由于这个机制，如果两个Bean都相互引用，都采用构造函数注入方式，就会发生类似于线程死锁的循环依赖问题。
```xml
    <!--5.循环依赖注入-->
    <bean id="boss1" class="com.vtstar.entity.Boss">
        <constructor-arg index="0" value="Tom"/>
        <constructor-arg index="1" ref="car4"/>
        <constructor-arg index="2" value="20"/>
    </bean>
 
    <bean id="car4" class="com.vtstar.entity.Car">
        <constructor-arg index="0" value="232.9"/>
        <constructor-arg index="1" ref="boss1"/>
    </bean>
```
控制台输出的结果：（这就是采用循环注入方式产生最大的问题）
![mskt_30](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_30_%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E7%9A%84%E9%97%AE%E9%A2%98.png)
如何解决这种问题？`将相互依赖的两个Bean中的其中一个Bean采用Setter注入的方式即可`。
```xml
    <!--5.循环依赖注入-->
    <bean id="boss1" class="com.vtstar.entity.Boss">
        <constructor-arg index="0" value="Tom"/>
        <constructor-arg index="1" ref="car"/>
    </bean>
 
    <bean id="car" class="com.vtstar.entity.Car">
        <property name="brand" value="保时捷 k3"/>
        <property name="maxSpeed" value="100"/>
        <property name="price" value="30.1"/>
    </bean>
```

#### 构造函数注入方式：
**优点**：</br>
- 1.构造函数可以保证一些重要的属性在bean实例化的时候就设置好，`避免因为一些重要的属性没有提供而导致一个无用的Bean 实例情况`
- 2.不需要为每个属性提供Setter方法，`减少了类的方法个数`
- 3.可以更好的封装类变量，不需要为每个属性提供Setter方法，`避免外部错误的调用`</br>

**缺点：**</br>
- 1.如果一个类属性太多，那么构造函数的参数将变成一个庞然大物，`可读性较差`
- 2.`灵活性不强`，在有些属性是可选的情况下，如果通过构造函数注入，也需要为可选的参数提供一个null值
- 3.如果有多个构造函数，则需要`考虑配置文件和具体构造函数匹配歧义`的问题，配置上相对复杂
- 4.构造函数`不利于类的继承和拓展`，因为子类需要引用父类复杂的构造函数
- 5.构造函数注入有时会造成`循环依赖`的问题

### 三. 工厂注入
既然需要一个工厂，那么我们需要创建一个CarFactory类
```java
package com.vtstar.ioc;
 
import com.vtstar.entity.Car;
 
/**
 * @ClassName CarFactory
 * @Description TODO
 * @Author XiaoWu
 * @Date 2018/9/6 13:55
 * @Version 1.0
 **/
public class CarFactory {
    /*
     * @methodName createHongQiCar
     * @Description 创建红旗轿车制造工厂
     * @Date 2018/9/6 13:58
     * @Param []
     * @return com.vtstar.entity.Car
     **/
    public Car createHongQiCar(){
        Car car = new Car();
        car.setBrand("红旗H1");
        System.out.println("这里是非静态工厂的创建..." + car.getBrand());
        return car;
    }
 
    /*
     * @methodName createDaZhongCar
     * @Description 创建大众汽车制造工厂
     * @Date 2018/9/6 14:02
     * @Param []
     * @return com.vtstar.entity.Car
     **/
    public static Car createDaZhongCar(){
        Car car = new Car();
        car.setBrand("大众GoGoGo");
        System.out.println("这里是静态工厂的创建..." + car.getBrand());
        return car;
    }
}
```
工厂注入方式分为 静态工厂和非静态工厂，相关Spring配置如下：
```xml
    <!--非静态注入工厂方法-->
    <bean id="carFactory" class="com.vtstar.ioc.CarFactory"/>
    <bean id="car5" factory-bean="carFactory" factory-method="createHongQiCar"/>
 
    <!--静态注入工厂方法-->
    <bean id="car6" class="com.vtstar.ioc.CarFactory" factory-method="createDaZhongCar"/>
```

### 四.基于注解的注入
在介绍注解注入的方式前，先简单了解bean的一个属性autowire，autowire主要有三个属性值：constructor，byName，byType。
- constructor：通过构造方法进行自动注入，spring会匹配与构造方法参数类型一致的bean进行注入，如果有一个多参数的构造方法，一个只有一个参数的构造方法，在容器中查找到多个匹配多参数构造方法的bean，那么spring会优先将bean注入到多参数的构造方法中。
- byName：被注入bean的id名必须与set方法后半截匹配，并且id名称的第一个单词首字母必须小写，这一点与手动set注入有点不同。
- byType：查找所有的set方法，将符合符合参数类型的bean注入。

主要有四种注解可以注册bean，每种注解可以任意使用，只是语义上有所差异：
- @Component：可以用于注册所有bean
- @Repository：主要用于注册dao层的bean
- @Controller：主要用于注册控制层的bean
- @Service：主要用于注册服务层的bean

描述依赖关系主要有两种：
- @Resource：java的注解，默认以byName的方式去匹配与属性名相同的bean的id，如果没有找到就会以byType的方式查找，如果byType查找到多个的话，使用@Qualifier注解（spring注解）指定某个具体名称的bean。
```java
@Resource
@Qualifier("userDaoMyBatis")
private IUserDao userDao;

public UserService(){
```
- @Autowired：spring注解，默认是以byType的方式去匹配与属性名相同的bean的id，如果没有找到，就通过byName的方式去查找
```java
@Autowired
@Qualifier("userDaoJdbc")
private IUserDao userDao;
```