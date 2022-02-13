# hashcode(), eqauls()生成算法、方法怎么重写

**答:**
```java
@Override
public boolean equals(Object object) {
//    //测试检测的对象是否为空，是就返回false
        if (object == null) {
            return false;
        }

//使用==操作符检查“参数是否为这个对象的引用”：如果是对象本身，则直接返回，拦截了对本身调用的情况，算是一种性能优化。
        if (object == this) {
            return true;
        }

//使用instanceof操作符检查“参数是否是正确的类型”：如果不是，就返回false
// 在 equals() 中最好使用 getClass 进行类型判断,
        if (!(object instanceof HashTest)) {
            return false;
        }
或者
 //测试两个对象所属的类是否相同，否则返回false
         if (this.getClass() != object.getClass()){
             return false
         }

       //对object进行类型转换以便和类A的对象进行比较
                A other=(A)otherObject;

        if (object.getI() == this.getI()) {
            return true;
        }
        return false;
}

@Override
public int hashCode() {
        return id.hashCode();
    }

或者
@Override
 public int hashCode() {
        int hash = 17;
        hash = hash * 31 + getName().hashCode();
        return hash;
    }

/*或者有更简单的方法
@Override
public int hashCode()
{
    return Object.hashCode(属性A，属性B，属性C，……);
}
*/

/*关于hashCode()计算过程中，为什么使用了数字31，主要有以下原因：
1、使用质数计算哈希码，由于质数的特性，它与其他数字相乘之后，计算结果唯一的概率更大，哈希冲突的概率更小。
2、使用的质数越大，哈希冲突的概率越小，但是计算的速度也越慢；31是哈希冲突和性能的折中，实际上是实验观测的结果。
3、JVM会自动对31进行优化：31 * i == (i << 5) – i
*/
```
**需要注意的地方：**</br>
- 我们在覆写 equals() 方法时，一般都是推荐使用 getClass 来进行类型判断，不是使用 instanceof

>原因：我们都清楚 instanceof 的作用是判断其左边对象是否为其右边类的实例，返回 boolean 类型的数据。
可以用来判断继承中的子类的实例是否为父类的实现。注意后面这句话：可以用来判断继承中的子类的实例是否为父类的实现，正是这句话在作怪。

- 覆盖equals时一定要覆盖hashCode
- equals函数里面一定要是Object类型作为参数
- equals方法本身不要过于智能，只要判断一些值相等即可

**特性：**</br>
如果两个对象equals，那么它们的hashCode必然相等</br>
但是hashCode相等，equals不一定相等。




**先看一下如果不被重写（原生）的hashCode和equals是什么样的？**
- 1.不被重写（原生）的hashCode值是根据内存地址换算出来的一个值。
```java
public native int hashCode();
```
- 2.不被重写（原生）的equals方法是严格判断一个对象是否相等的方法,包括地址值。（object1 == object2）
```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```
>**我们来看一下Object.hashCode的通用约定（摘自《Effective Java》第45页）**

>- 在应用程序执行期间，只要对象的equals方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，hashCode方法都必须始终返回同一个值。在一个应用程序与另一个程序的执行过程中，执行hashcode所返回的值可以不一致。
>- 如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中的hashCode方法必须产生同样的整数结果。
>- 如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中的hashCode方法，则不一定要求hashcode方法必须产生不同的结果。但程序员应该明白，对于不相等的对象产生截然不同的整数结果，有可能提高散列表（hash table）的性能。

`如果只重写了equals方法而没有重写hashCode方法的话，则会违反约定的第二条：相等的对象必须具有相等的散列码(hashCode)`</br>

> 根据类的eqauls方法，两个截然不同的实例在逻辑上有可能是相等的，但是根据Object类的hashCode方法，他们仅仅是两个没有任何共同之处的对象。因此，对象的hashCode方法返回两个看起来是随机的整数，而不是根据第二个约定所要求的那样，返回两个相等的整数。

**所以eqauls和hashCode方法的设计应该满足如下:**
- (1)当`obj1.equals(obj2)`为true时，`obj1.hashCode() == obj2.hashCode()`必须为true
- (2)当`obj1.hashCode() == obj2.hashCode()为false`时，`obj1.equals(obj2)`必须为false


如果不重写equals，那么比较的将是对象的引用是否指向同一块内存地址，重写之后目的是为了比较两个对象的value值是否相等。

这样如果我们对一个对象重写了euqals，意思是只要对象的成员变量值都相等那么euqals就等于true，但如果不重写hashcode，那么我们再new一个新的对象，
当原对象和新对象内容相同，两者的hashcode却是不一样的，由此将产生了理解的不一致导致混淆。
>例如如在存储散列集合时（如HashMap类），将会依据不同的hashCode存储了两个内容一样的对象；</br>
亦或者根据新对象为key去map中取之前存储key为旧对象的值时，却得到null.</br>
>例如：如果只重写eqauls而不重写hashCode()，如下代码结果为null
>```java
>Map<PhoneNumber, String> m = new HashMap<>();
>m.put(new PhoneNumber(707, 867, 5309), "Jenny");
>system.out.println(m.get(PhoneNumber(new 707, 867, 5309)));
>```

**首先equals方法必须满足以下**:
- 自反性（x.equals(x)必须返回true）、
- 对称性（x.equals(y)返回true 时，y.equals(x)也必须返回true）、
- 传递性（x.equals(y)和y.equals(z)都返回true时，x.equals(z)也必须返回true）
- 一致性（当x和y引用的对象信息没有被修改时，多次调用x.equals(y)应该得到同样的返回值），
- 而且对于任何非null值的引用x，x.equals(null)必须返回false。

**实现高质量的equals方法的诀窍包括**：
- 使用==操作符检查"参数是否为这个对象的引用"；
- 使用 instanceof操作符检查"参数是否为正确的类型"；
- 对于类中的关键属性，检查参数传入对象的属性是否与之相匹配；
- 编写完equals方法后，问自己它是否满足对称性、传递性、一致性；
- 重写equals时总是要重写hashCode；
- 不要将equals方法参数中的Object对象替换为其他的类型，在重写时不要忘掉@Override注解。