只记录不熟的，更多见
[小林coding](https://xiaolincoding.com/interview/java.html#%E6%A6%82%E5%BF%B5)
[java笔记](https://mcnerzykwkel.feishu.cn/wiki/BwO0wlcRwiuVfPkGMTocox1qnye)
# 一.数据类型
基本数据类型，引用数据类型不赘述
## 1.BigDecimal
###  1.1为什么用bigDecimal 不用double ？
因为double有时不能准确表达小数（浮点数二进制）
而BigDecimal可以准确表达。
### 1.2常用api
#### 1.2.1构造方法
BigDecimal(double):可能不准确
BigDecimal(String):准确且能存很大范围的小数，
但在存0~10整数时不如valueof(因为0~10内的整数已经帮我们new好了)
BigDecimal.valueof(int,double):精确，但只能存到double范围内的数

#### 1.2.2常用方法
```Java
public BigDecimal add(BigDecimal value)    // 加法运算
public BigDecimal subtract(BigDecimal value)    // 减法运算
public BigDecimal multiply(BigDecimal value)    // 乘法运算
public BigDecimal divide(BigDecimal value)    // 除法运算
```
### 底层存储方式
从左向右将每一位ASCLL码存到数组中

## 2.包装类
### 2.1装箱和拆箱
#### 2.1.1定义
装箱（Boxing）和拆箱（Unboxing）是将基本数据类型和对应的包装类之间进行转换的过程。
```java
Integer i = 10;  //装箱(基本->包装)
int n = i;   //拆箱(包装->基本)
```
#### 2.1.2弊端
自动装箱有一个问题，那就是在一个循环中进行自动装箱操作的情况，如下面的例子就会**创建多余的对象，影响程序的性能。**
```java
Integer sum = 0; for(int i=1000; i<5000; i++){   sum+=i; } 
```

上面的代码sum+=i可以看成sum = sum + i，但是+这个操作符不适用于Integer对象，首先sum进行自动拆箱操作，进行数值相加操作，最后发生自动装箱操作转换成Integer对象。其内部变化如下
```java
int result = sum.intValue() + i; Integer sum = new Integer(result); 
```

由于我们这里声明的sum为Integer类型，在上面的循环中会创建将近4000个无用的Integer对象，在这样庞大的循环中，会降低程序的性能并且加重了垃圾回收的工作量。因此在我们编程时，需要注意到这一点，正确地声明变量类型，避免因为自动装箱引起的性能问题。

### 2.2为什么要有包装类
1.对象封装：包装类就是基本类型的封装对象。对象封装有很多好处，可以把属性也就是数据跟处理这些数据的方法结合在一起，方便使用。
2.泛型中的应用（集合中的应用）：泛型只能使用引用类型，而不能使用基本类型。因此，如果要在泛型中使用基本类型，必须使用其包装类。
3.转换中的应用：基本类型和引用类型不能直接进行转换，必须使用包装类来实现。例如，将一个int类型的值转换为String类型，必须首先将其转换为Integer类型，然后再转换为String类型。

### 2.3为什么保留基本类型
1.读写效率高：包装类是引用类型，对象的引用和对象本身是分开存储的，而对于基本类型数据，变量对应的内存块直接存储数据本身。
2.存储效率：以Integer为例：在64位JVM上，在开启引用压缩的情况下，一个Integer对象占用16个字节的内存空间，而一个int类型数据只占用4字节的内存空间，前者对空间的占用是后者的4倍。（引用类型除了数据本身外，还有对象头等额外信息。内存占用比基本类型大）

### 2.4Integer缓存
Java的Integer类内部实现了一个静态缓存池，用于存储特定范围内的整数值对应的Integer对象。

默认情况下，这个范围是-128至127。当通过Integer.valueOf(int)方法创建一个在这个范围内的整数对象时，并不会每次都生成新的对象实例，而是复用缓存中的现有对象，会直接从内存中取出，不需要新建一个对象。

# 二.面向对象

## 1.java面向对象三大特性
Java面向对象的三大特性包括：封装、继承、多态：

- 封装：封装是指将对象的属性（数据）和行为（方法）结合在一起，对外隐藏对象的内部细节，仅通过对象提供的接口与外界交互。封装的目的是增强安全性和简化编程，使得对象更加独立。

- 继承：继承是一种可以使得子类自动共享父类数据结构和方法的机制。它是代码复用的重要手段，通过继承可以建立类与类之间的层次关系，使得结构更加清晰。

- 多态：多态是指允许不同类的对象对同一消息作出响应。即同一个接口，使用不同的实例而执行不同操作。多态性可以分为编译时多态（重载）和运行时多态（重写）。它使得程序具有良好的灵活性和扩展性。

## 2.多态：
### 2.1多态体现：
- 方法重载：
	方法重载是指同一类中可以有多个同名方法，它们具有不同的参数列表（参数类型、数量或顺序不同）。虽然方法名相同，但根据传入的参数不同，编译器会在编译时确定调用哪个方法。
	
- 方法重写:
	是指子类能够提供对父类中同名方法的具体实现。在运行时，JVM会根据对象的实际类型确定调用哪个版本的方法。这是实现多态的主要方式。
- 接口与实现：
	多态也体现在接口的使用上，多个类可以实现同一个接口，并且用接口类型的引用来调用这些类的方法。这使得程序在面对不同具体实现时保持一贯的调用方式。

- 向上转型和向下转型：
	在Java中，可以使用父类类型的引用指向子类对象，这是向上转型。通过这种方式，可以在运行时期采用不同的子类实现。
### 2.2多态运行特点
调用**成员变量**时：编译看左边，运行看左边
调用**成员方法**时：编译看左边，运行看右边

```Java
Fu f = new Zi()；
//编译看左边的父类中有没有name这个属性，没有就报错
//在实际运行的时候，把父类name属性的值打印出来
System.out.println(f.name);
//编译看左边的父类中有没有show这个方法，没有就报错
//在实际运行的时候，运行的是子类中的show方法
f.show();
```
如果子类没有这个方法呢？
到虚方法表里找：调用最后被覆盖的那个方法
**虚方法表**：
a.每个类都有一个虚方法表，记录该类非 `private`、非 `static`、非 `final` 的实例方法
b.子类的虚方法表为：父类的虚方法表+自己的虚方法。
c.方法重写的本质：覆盖虚方法表

### 2.3转型
多态的转型分为向上转型（自动转换）与向下转型（强制转换）两种。

**向上转型（自动转换）**：

多态本身是子类类型向父类类型向上转换（自动转换）的过程，这个过程是默认的。当父类引用指向一个子类对象时，便是向上转型。

**向下转型（强制转换）**：

父类类型向子类类型向下转换的过程，这个过程是强制的。将父类引用转为子类引用，可以使用强制类型转换的格式。
不能转到自己同级或低级的类

**转型时的异常**：

#### 2.3.1 instanceof关键字

为了避免ClassCastException的发生，Java提供了 `instanceof` 关键字，给引用变量做类型的校验：

```Java
变量名 instanceof 数据类型 
如果变量属于该数据类型或者其子类类型，返回true。
如果变量不属于该数据类型或者其子类类型，返回false。
```
**instanceof新特性**：
JDK14的时候提出了新特性，把判断和强转合并成了一行。
```Java
//先判断a是否为Dog类型，如果是，则强转成Dog类型，转换之后变量名为d
//如果不是，则不强转，结果直接是false
if(a instanceof Dog d){
    d.lookHome();
}else if(a instanceof Cat c){
    c.catchMouse();
}else{
    System.out.println("没有这个类型，无法转换");
}
```
## 3.抽象类，抽象方法
### 1.相关概念：
#### 1.1定义
如果父类知道子类需要使用的方法，但是不知道怎么实现，可以使用抽象的思想实现。
- **抽象方法** ： 没有方法体的方法
- **抽象类**：包含抽象方法的类
**抽象方法**：
```Java
修饰符 abstract 返回值类型 方法名 (参数列表);
示例：
public abstract void run();
```
**抽象类**：

```Java
abstract class 类名字 { 
}
示例：
public abstract class Animal {
    public abstract void run();
}
```

#### 1.2注意
- *抽象类不能创建对象*，如果创建，编译无法通过而报错。只能创建其非抽象子类的对象
- 抽象类中，可以有构造方法，是供子类创建对象时，初始化父类成员使用的
- 抽象类中，可以有构造方法、实例方法、静态方法和成员变量
- 抽象类中，*不一定包含抽象方法，但是有抽象方法的类必定是抽象类*
- 抽象类的子类，必须重写抽象父类中所有的抽象方法，否则子类也必须定义成抽象类，编译无法通过而报错

## 4.接口
### 4.1JDK7及之前的接口
接口是更加彻底的抽象，JDK7之前，包括JDK7，接口中全部是抽象方法。接口同样是**不能创建对象**的。
JDK7之前，接口中的**只有**包含：抽象方法和常量

接口中的抽象方法默认会自动加上`public abstract`修饰，程序员无需自己手写！
在接口中定义的成员变量默认会加上：`public static final`修饰，也就是说在接口中定义的成员变量实际上是一个**常量**，并且可以直接用接口名访问。

### 4.2接口的实现和继承
- 类与接口之间的关系是多实现的，一个类可以同时实现多个接口
	- **类实现接口的要求**：
		1. 必须重写实现的全部接口中所有抽象方法
		2. 如果一个类实现了接口，但是没有重写完全部接口的全部抽象方法，这个类也必须定义成抽象类
- 接口与接口之间是可以多继承的，也就是一个接口可以同时继承多个接口

### 4.3JDK8及以后接口的特点
JDK8以后接口中可以定义有方法体的方法。
- **允许在接口中定义默认方法，需要用default修饰**：
	- 默认方法不是抽象方法，不强制被重写，但如果重写，需要去掉default关键字
	- public可以省略，但是default不能省略
	- 如果实现了多个接口，且多个接口中存在名字相同的默认方法，子类就必须对该方法进行重写
- **允许在接口中定义静态方法，需要用static修饰**：
JDK9以前，接口中只能定义public方法，JDK9以后，还可以在接口中定义private方法。
private 方法用来给默认方法服务

## 5.内部类
将一个类A定义在另一个类B里面，里面的那个类A就称为**内部类**，B则称为**外部类**。
具体见[java笔记](https://mcnerzykwkel.feishu.cn/wiki/BwO0wlcRwiuVfPkGMTocox1qnye)九.2
### 5.1 内部类的分类

1. **成员内部类**，类定义在了成员位置 (类中方法外称为成员位置，无static修饰的内部类)
    
2. **静态内部类**，类定义在了成员位置 (类中方法外称为成员位置，有static修饰的内部类)
    
3. **局部内部类**，类定义在方法内
    
4. **匿名内部类**，没有名字的内部类，可以在方法中，也可以在类中方法外
## 6.关键字

### 6.1static
#### 6.1.1静态变量
静态变量（也称为类变量）是在类中使用static关键字声明的变量。它们属于类而不是任何具体的对象。主要的特点：
- 共享性：所有该类的实例共享同一个静态变量。如果一个实例修改了静态变量的值，其他实例也会看到这个更改。
- 初始化：静态变量在类被加载时初始化，只会对其进行一次分配内存。
- 访问方式：静态变量可以直接通过类名访问，也可以通过实例访问，但推荐使用类名。

#### 6.1.2静态方法
静态方法是在类中使用static关键字声明的方法。类似于静态变量，静态方法也属于类，而不是任何具体的对象。主要的特点：
- 无实例依赖：静态方法可以在没有创建类实例的情况下调用。(常用于工具类)
- 访问静态成员：静态方法可以直接调用其他静态变量和静态方法，但不能直接访问非静态成员。也不能用this
- 多态性：静态方法不支持重写（Override），但可以被隐藏（Hide）。

#### 6.1.3修饰代码块
静态代码块在类加载时执行，且只执行一次（优于对象构造方法），用于初始化静态变量或执行类级别的预处理操作。

多个静态代码块按定义顺序执行，且先于非静态代码块和构造方法。

#### 6.1.4修饰内部类
静态内部类不依赖于外部类的实例，可以独立存在，不能直接访问外部类的非静态成员（需通过外部类实例访问）。

当内部类与外部类的实例无关时使用，避免内部类持有外部类的引用导致的内存泄漏。
### 6.2final
#### Java 中 final 作用是什么？
final关键字主要有以下三个方面的作用：用于修饰类、方法和变量。
- 修饰类：当final修饰一个类时，表示这个类**不能被继承**，是类继承体系中的最终形态。
- 修饰方法：用final修饰的方法**不能在子类中被重写**。
- 修饰变量：当final修饰基本数据类型的变量时，该变量一旦被赋值就不能再改变。对于引用数据类型，final修饰意味着这个引用变量不能再指向其他对象，但对象本身的内容是可以改变的。

### 6.3权限修饰符
| public   | protected | 默认  | private |     |
| -------- | --------- | --- | ------- | --- |
| 同一类中     | √         | √   | √       | √   |
| 同一包中的类   | √         | √   | √       |     |
| 不同包的子类   | √         | √   |         |     |
| 不同包中的无关类 | √         |     |         |     |

## 7.代码块
1.**局部代码块**
- 用大括号把一段代码括起来，用以控制变量的生命周期
```Java
public class Test{
    public static void main(String[] args){
        {
            int num = 10; //只在包裹它的第一个大括号中有效
        }
        num = 20; //报错，因为num未定义
    }
}
```
2.**构造代码块**
- 写在成员位置的代码块
- 可以把多个构造方法中的重复代码抽取出来
- 执行时机：在创建本类对象的时候会先执行构造代码块再执行构造方法
```Java
public class Student{
        private String name;
        {
                System.out.println("构造代码块");
        }
        public Student(){
                System.out.println("空参构造");
        }
}
```
- 每次创建对象都会先打印构造代码块（创建几次打印几次），再打印空参构造
3.**静态代码块**
- 格式：static{}
- 特点：需要通过关键字修饰，随着类的加载而加载，并且自动触发，只执行一次
- 使用场景：在类加载时，做一些数据初始化时使用
```Java
public class Student{
        private String name;
        static{
                System.out.println("构造代码块");
        }
        public Student(){
                System.out.println("空参构造");
        }
}
```
- 只有第一次创建对象时打印`构造代码块`，后面创建对象不会打印



# 三.java集合
## 1.泛型
### 1.1什么是泛型
泛型就是数据类型的泛指，可以泛指任何引用数据类型，将来可以被任何引用数据类型替代。

### 1.2为什么需要泛型
a.适用于多种数据类型执行相同的代码
```java
private static int add(int a, int b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static float add(float a, float b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static double add(double a, double b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}
```
如果没有泛型，要实现不同类型的加法，每种类型都需要重载一个add方法；通过泛型，我们可以复用为一个方法：
```java
private static <T extends Number> double add(T a, T b) {
    System.out.println(a + "+" + b + "=" + (a.doubleValue() + b.doubleValue()));
    return a.doubleValue() + b.doubleValue();
}
```

b.泛型中的类型在使用时指定，不需要强制类型转换（类型安全，编译器会检查类型）
看下这个例子：
```java
List list = new ArrayList();
list.add("xxString");
list.add(100d);
list.add(new Person());
```
我们在使用上述list中，list中的元素都是Object类型（无法约束其中的类型），所以在取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易出现java.lang.ClassCastException异常。

引入泛型，它将提供类型的约束，提供编译前的检查：
```java
List<String> list = new ArrayList<String>();

// list中只能放String, 不能放其它类型的元素
```

### 1.3泛型的通配符
**通配符格式**
- `？ extends E`：表示可以传递E或者E的所有子类类型
    
- `？ super E`：表示可以传递E或E的所有父类类型


## 2.单列集合
![[f9b5a436fdc6f16ff6b585933745f5b7.jpg]]
### 2.1Collection
单例集合的顶层接口

### 2.2List
**特点**
- 存取有序：存和取的顺序是一致的
- 可以重复：可以存放重复的数据
- 有索引：用户可以根据索引获取数据，或操作指定索引处的数据

#### 2.2.1ArrayList
**特点**：
- 基于动态数组实现，它允许快速的随机访问，即通过索引访问元素的时间复杂度为 O (1)。
- 在添加和删除元素时，如果操作位置不是列表末尾，可能需要移动大量元素，性能相对较低。
- 适用于需要频繁随机访问元素，而对插入和删除操作性能要求不高的场景，如数据的查询和展示等。

**扩容机制**：
ArrayList在添加元素时，如果当前元素个数已经达到了内部数组的容量上限，就会触发扩容操作。
1.计算新数组大小
	1.加一个元素：
		a.第一次:创建长度为10的数组
		b.其它：扩容1.5倍
	2.加多个元素：
		a.第一次添加：
			- 添加元素<=10：创建长度为10的数组
			- 添加元素>10：创建个数为添加个数的数组
		b.其它：
			- 扩容1.5倍存的下：扩容1.5倍
			- 扩容1.5倍存不下：扩容至（原来长度+要添加元素的个数）
2.创建新的数组：根据计算得到的新容量，创建一个新的更大的数组。
3.将元素复制：将原来数组中的元素逐个复制到新数组中。
4.更新引用：将ArrayList内部指向原数组的引用指向新数组。
5.完成扩容：扩容完成后，可以继续添加新元素。

#### 2.2.2LinkedList
**特点**
- LinkedList基于双向链表实现，在插入和删除元素时，只需修改链表的指针，不需要移动大量元素，时间复杂度为 O (1)。
- 但随机访问元素时，需要从链表头或链表尾开始遍历，时间复杂度为 O (n)。
- 适用于需要频繁进行插入和删除操作的场景，如队列、栈等数据结构的实现，以及需要在列表中间频繁插入和删除元素的情况。

### 2.3Set
**特点**
- 存取是否有序取决于实现类
- 不可以存储重复元素
- 没有索引：不能使用普通for循环遍历

#### 2.3.1HashSet
**特点**
- 存取无序

**底层实现**
- **JDK8以前**
创建一个默认长度16，默认加载因子0.75的数组，数组名为table。
根据元素的哈希值跟数组的长度计算出应存入的位置：
```Java
int index = (数组长度 - 1) & 哈希值；
```
新元素存入数组，**老元素挂在新元素下面**：数组 + 链表
![[Pasted image 20260427145626.png]]

- **JDK8以后**
**新元素直接挂在老元素的下面**：
- 节点个数少于等于8个：数组 + 链表
- 节点个数多于8个：数组 + 红黑树
![[Pasted image 20260427145637.png]]
> HashSet集合存储自定义类型元素，要想实现元素的唯一，要求必须重写hashCode方法和equals方法

#### 2.3.2LinkedHashSet
**特点**
- 存取有序

**底层实现**
- 底层数据结构依然是哈希表，只是每个元素又多了一个双向链表用来记录存储的顺序
![[Pasted image 20260427145650.png]]

#### 2.3.3TreeSet
**特点**
- 可以将元素按规则排序
- 底层用红黑树存储

**如何排序**
1.自然排序：TreeSet()无参构造
- 对于基本数据类型：Integer，Double，默认按照从小到大进行升序排序
- 对于字符、字符串类型：按照字符在ASCII码表中的数字升序排序
- 对于引用类型：实现Comparable接口并重写`compareTo(T o)`方法

以引用类型为例
```Java
public class Student implements Comparable<Student>{
    private String name;
    private int age;
        ...
        //this:表示当前要添加的元素
        //o：表示已经在红黑树存在的元素
        //返回值：
        //负数：认为要添加的元素是小的，放左边
        //正数：认为要添加的元素是大的，放右边
        //0：认为要添加的元素已存在，舍弃。
        //this-o代表升序
        //o-this代表降序
    @Override
    public int compareTo(Student o) {
        //按照对象的年龄进行排序
        //主要判断条件: 按照年龄从小到大排序
        int result = this.age - o.age;
        //次要判断条件: 年龄相同时，按照姓名的字母顺序排序
        result = result == 0 ? this.name.compareTo(o.getName()) : result;
        return result;
    }
}
```
然后使用无参构造，向TreeSet里传自定义对象，其排序顺序就按照compareTo里写的那样

2.比较器排序：TreeSet(Comparator)有参构造
- 按重写的Comparator里的compare方法排序
```Java
//创建集合对象
TreeSet<Teacher> ts = new TreeSet<>(new Comparator<Teacher>() {
    @Override
    public int compare(Teacher o1, Teacher o2) {
        //o1表示现在要存入的那个元素
        //o2表示已经存入到集合中的元素
        //主要条件
        int result = o1.getAge() - o2.getAge();
        //次要条件
        result = result == 0 ? o1.getName().compareTo(o2.getName()) : result;
        return result;
    }
});
```


## 3.双列集合
![[Pasted image 20260422194806.png]]

### 3.1Map
**特点**
- 一次需要存一对数据，分别是键和值
- 键不能重复，值可以重复
- 每一个键对应着一个值
- 键+值这个整体称为“键值对”或“Entry对象”

**常用api**

| 方法名                                 | 说明                 |
| ----------------------------------- | ------------------ |
| V put(K key,V value)                | 添加元素               |
| V remove(Object key)                | 根据键删除键值对元素         |
| void clear()                        | 移除所有的键值对元素         |
| boolean containsKey(Object key)     | 判断集合是否包含指定的键       |
| boolean containsValue(Object value) | 判断集合是否包含指定的值       |
| boolean isEmpty()                   | 判断集合是否为空           |
| int size()                          | 集合的长度，也就是集合中键值对的个数 |
| V get(Object key)                   | 根据键获取值             |
| Set`<K>` keySet()                   | 获取所有键的集合           |
| Collection`<V>` values()            | 获取所有值的集合           |
| Set<Map.Entry<K,V>> entrySet()      | 获取所有键值对对象的集合       |
**怎么遍历**
1.调map.KeySet()后，遍历set，对每个key,调用map.get(key)。
2.调map.entrySet()后，遍历entryset，对每个set,调用getKey(),getValue()
3.map.forEach((key,value)->...)(底层调用第二种方法)

#### 3.1.1HashMap
##### **特点**
- 特点都是由键决定的：无序、不重复、无索引
- HashMap跟HashSet底层原理是一样的，都是哈希表结构
- 依赖hashCode方法和equals方法保证键的唯一
    - 如果键要存储的是自定义对象，需要重写hashCode和equals方法

##### **实现原理**
在 JDK 1.7 版本之前，
HashMap 数据结构是数组和链表，HashMap通过哈希算法将元素的键（Key）映射到数组中的槽位（Bucket）。如果多个键映射到同一个槽位，它们会以链表的形式存储在同一个槽位上，因为链表的查询时间是O(n)，所以冲突很严重，一个索引上的链表非常长，效率就很低了。
![[Pasted image 20260422200122.png|606]]

所以在 JDK 1.8 版本的时候做了优化：
扩容：
- 扩容时机
	- 键值对的数量超过了哈希表长度 * 装载因子(0.75)
	- 某个桶的链表长度>=8，且哈希表长度<64
- 扩容过程
	- 创建一个长度为原来两倍的哈希表
	- 逐个将键值对迁移到新哈希表

树化：
- 树化时机：
	- 当某个桶的链表长度 ≥ 8（TREEIFY_THRESHOLD）且哈希表数组长度 ≥ 64（MIN_TREEIFY_CAPACITY）时，会把链表转换为红黑树，把该桶的查找时间复杂度从 O(n) 降低到 O(log n)；
	- 如果哈希表长度不到64，只会触发扩容
- 树变链表，
	- 在 resize() 过程中，若某个桶的节点数 ≤ 6（UNTREEIFY_THRESHOLD），红黑树会被退化回链表。
![[Pasted image 20260422200415.png]]

##### **为什么用红黑树不用AVL树**
1.AVL 树是严格平衡的二叉树，要求任意节点的左右子树高度差不超过 1，插入 / 删除时会触发大量旋转操作（左旋、右旋、双旋）
2.红黑树仅保证黑色高度平衡（不是严格的节点高度平衡），插入/删除时，旋转次数远少于 AVL 树。
3.HashMap“增删” 和 “查找” 的频率几乎持平,尽管红黑树查询效率略低于AVL树，但增删的开销远低于 AVL 树。所以再HashMap这种情况下实际效率更高

##### get,put方法
[get,put方法](https://xiaolincoding.com/interview/collections.html#%E5%9C%A8-java-%E7%9A%84-hashmap-%E4%B8%AD-get%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0%E7%9A%84%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84)

##### 为什么哈希表的长度为$2^n$
1.保证 hash & (length - 1) 等价于取模，用位运算实现高效寻址；
  a.索引计算用的是位运算：hash & (length - 1)。（为什么用位运算：位运算比取模运算快）
  b.让length=$2^n$,可以使length-1低n位均为1，达到取模的效果
如果不为$2^n$：
```
01100100  (hash = 100)
& 00001110  (length-1 = 14)
= 00000100  (结果 = 4)

01100101  (hash = 101)
& 00001110  (length-1 = 14)
= 00000100  (结果还是 4！)
```
这不仅导致空间浪费，还会增大hash碰撞的结果

2.让 length - 1 的二进制全 1，接住 hash 值的均匀分布，减少碰撞；

  HashMap 会对 key 的原始 hashCode 做**扰动处理**（比如 JDK1.8 里是 hash = (h = key.hashCode()) ^ (h >>> 16)），目的是让 hash 值的二进制位尽可能均匀分布。

  但只有当 length - 1 的二进制是全 1 时，才能 “接住” 这些均匀分布的位。

3.为扩容优化铺路，不用重新算 hash，仅通过高位判断就能快速确定新索引。
  扩容其实不用重新计算新的hash值，只用判断新高位（n）=（hash&oldcap）为1还是0，
  为1：移动$2^n$位
  为0：不用动
为什么：
  每次扩容扩大两倍，新位置=hash&(oldcap * 2-1)，oldcap右移一位，与的低几位不变，若新高位为1，则新位置=原位置+$2^n$,若为0，则说明不变
  例：从16扩展为32
![[Pasted image 20260422204256.png]]
![[Pasted image 20260422204325.png]]

总结：便于位运算，降低哈希冲突，便于扩容
#### 3.1.2LinkedHashMap
**特点**
- 有序：由双向链表保证

#### 3.1.3TreeMap
**特点**
- 可排序：底层是红黑树(如何排序与TreeSet类似)

# 四.异常
## 1.异常分类
![[Pasted image 20260422205437.png]]
Java 的异常体系主要基于 Throwable 及其子类。Throwable 有两个重要的子类：Error 和 Exception，它们分别代表了不同类型的异常情况。

- Error (错误)： 表示运行环境的错误，错误是程序无法处理的严重问题，如虚拟机错误、动态链接库失效等。程序不应该尝试捕获这类错误。例如，OutOfMemoryError、StackOverflowError 等。
- Exception (异常)： 表示程序本身可以处理的异常情况。异常分为两大类：
	- **非运行时异常**（受检异常，Checked Exception）： 这类异常在编译时就必须被捕获或者声明抛出。
		- 它们通常是外部错误，如文件不存在(FileNotFoundException)、类未找到 (ClassNotFoundException) 等。非运行时异常强制程序员处理这些可能出现的问题，增强了程序的健壮性。
	- **运行时异常**（非受检异常，Unchecked Exception 或 RuntimeException）： 这类异常特指 RuntimeException 及其子类。它与 Error 一起构成了 Java 中的非受检异常家族。
		- 运行时异常由程序逻辑错误导致，如空指针访问 (NullPointerException)、数组越界 (ArrayIndexOutOfBoundsException) 等。运行时异常是不需要在编译时强制捕获或声明的。

## 2.异常处理方法
### 1.try...catch...finally
```java
try {
    // 可能抛出异常的代码
} catch (ExceptionType e) {
    // 处理异常的逻辑
} finally {
    // 无论是否发生异常，都会执行的代码
}
```
### 2.手动抛出异常
throw**用在方法内**，用来抛出一个异常对象，将这个异常对象传递到调用者处，并结束当前方法的执行。

```Java
throw new 异常类名(参数);

例如：
throw new NullPointerException("要访问的arr数组不存在");
```

### 3.throws关键字
关键字**throws**运用于方法声明之上，用于表示当前方法不处理异常，而是提醒该方法的调用者来处理异常（抛出异常）。

```Java
修饰符 返回值类型 方法名(参数) throws 异常类名1,异常类名2…{   }

例如：
public static double division(double x, double y) {
    if (y == 0) {
        throw new ArithmeticException("除数不能为0");
    }
    return x / y;
}
```

## 3.Throwable中的常用方法

| public void printStackTrace() | 打印异常的详细信息，在底层是利用`System.err.println`进行输出，仅仅打印信息，不会停止程序的执行。包含了异常的类型、异常的原因 和 异常出现的位置，在开发和调试阶段，都得使用printStackTrace |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| public String getMessage()    | 获取发生异常的原因。示给用户的时候，就提示错误原因                                                                                       |
| public String toString()      | 获取异常的类型和异常描述信息                                                                                                  |
|                               |                                                                                                                 |

> System.err.println表示把信息以红色字体输出到控制台，基本不用。

## 4.自定义异常
**继承关系**：
    1. 编译时异常：继承`java.lang.Exception`
    2. 运行时异常：继承`java.lang.RuntimeException`
```Java
public class LoginException extends Exception {
    /**
     * 空参构造
     */
    public LoginException() {
    }

    /**
     * @param message 表示异常提示
     */
    public LoginException(String message) {
        super(message);
    }
}
```

# 五.反射，动态代理，注解
## 1.反射
### 1.获取字节码文件对象
**字节码文件**：Java中的类经过编译后可以得到一个`class`后缀的文件，这个在HelloWorld案例中就可以看到。由于idea会自动帮我们编译，所以开发中感受不到class文件的存在。

字节码文件对象是属于类**Class**的对象，有三种获取方式：
1. 源代码阶段：编写Java代码的阶段，程序员使用文本编辑器或集成开发环境（IDE）编写Java源代码文件
    1. 使用`Class.forName("类的全类名")`
2. 加载阶段：当Java程序运行时，JVM（Java虚拟机）需要将字节码文件加载到内存中
    1. 使用`类名.class`
3. 运行阶段：程序实际执行的阶段，JVM解释或编译执行字节码
    1. 使用`对象.getClass()`
    2. 同一个类的不同对象获取到的是同一个字节码文件对象
### 2.获取构造方法（Constructor）
类的构造方法对象是属于**Constructor**类的对象：
有s:全部，无s：单个
有Declared:全部，无...：公共

|Class类中的方法|说明|
|---|---|
|Constructor[] getConstructors()|获得所有公共构造方法对象|
|Constructor[] getDeclaredConstructors()|获得所有的构造方法对象|
|Constructor`<T>` getConstructor(Class... parameterTypes)|获取单个公共构造方法对象|
|Constructor`<T>` getDeclaredConstructor(Class... parameterTypes)|获取单个构造方法对象|

Constructor类中用于创建对象的方法：

|方法名|说明|
|---|---|
|T newInstance(Object... initargs)|根据指定的构造方法创建对象|
|setAccessible(boolean flag)|设置为true表示取消访问检查|
|int getModifiers()|获取此构造方法的权限修饰符|

> 对于私有构造方法，如果要强行创建对象，必须先用setAccessible方法取消访问检查

### 3.获取成员变量(Field)

类的成员变量对象是属于**Field**类的对象：

|Class类中的方法|说明|
|---|---|
|Field[] getFields()|返回所有公共成员变量对象的数组|
|Field[] getDeclaredFields()|返回所有成员变量对象的数组|
|Field getField(String name)|返回单个公共成员变量对象|
|Field getDeclaredField(String name)|返回单个成员变量对象|

Field类中用于创建对象的方法：

|方法名|说明|
|---|---|
|void set(Object obj, Object value)|给指定成员变量赋值|
|Object get(Object obj)|获取指定成员变量的值|
|String getName()|获取指定成员变量的变量名|

### 4.获取成员方法(Method)

类的成员方法对象是属于**Method**类的对象：

|方法名|说明|
|---|---|
|Method[] getMethods()|返回所有公共成员方法对象的数组|
|Method[] getDeclaredMethods()|返回所有成员方法对象的数组|
|Method getMethod(String name, Class... parameterTypes)|返回单个公共成员方法对象|
|Method getDeclaredMethod(String name, Class... parameterTypes)|返回单个成员方法对象|

Method类中用于调用成员方法的方法：

`Object invoke(Object obj, Object...args)`：运行方法
- 参数一：用obj对象调用该方法
- 参数二：调用方法的传递参数（如果没有就不写）
- 返回值：方法的返回值（如果没有就不接收）

### 5.使用案例
```java
//1.获取字节码文件对象 
Class clazz = Class.forName(classname); 
//2.要先创建这个类的对象 
Constructor con = clazz.getDeclaredConstructor(); 
con.setAccessible(true); 
Object o = con.newInstance(); 
//3. 获取所有的成员变量 
Field[] fields = clazz.getDeclaredFields(); 
for (Field field : fields) { 
field.setAccessible(true); 
//获取成员变量的名字 
String name = field.getName(); 
//获取成员变量的值 
Object value = field.get(obj);
}

//4.获取方法的对象 
Method method = clazz.getDeclaredMethod(methodname);
 method.setAccessible(true); 
 //5.运行方法 
 method.invoke(o);
```

## 2.动态代理
为什么要代理：
动态代理可以无无侵入式的给方法增强功能，即如果想要给项目增加功能，通过动态代理可以在不动项目源码的情况下实现增加功能。

[如何实现动态代理](https://lvdkpt.blog.csdn.net/article/details/148934524?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7Ebaidujs_baidulandingword%7ECtr-3-148934524-blog-156390944.235%5Ev43%5Epc_blog_bottom_relevance_base5&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7Ebaidujs_baidulandingword%7ECtr-3-148934524-blog-156390944.235%5Ev43%5Epc_blog_bottom_relevance_base5&utm_relevant_index=5)
#todo 如何理解动态代理
## 3.注解
参考：[注解](https://blog.csdn.net/cjejwe/article/details/160362394?ops_request_misc=&request_id=&biz_id=102&utm_term=%E6%B3%A8%E8%A7%A3&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduweb~default-0-160362394.nonecase&spm=1018.2226.3001.4450)
### 1.注解是什么
注解本质是一个继承java.lang.annotation.Annotation的接口
```java
//定义一个注解
public @interface MyTag { 
}

//@interface是java的语法糖，编译后会变成
public interface MyTag extends java.lang.annotation.Annotation { 
}
```
**Annotation长什么样：**
```java
// java.lang.annotation.Annotation —— 所有注解的祖先
package java.lang.annotation;

public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();
    
    // 返回注解的类型
    Class<? extends Annotation> annotationType();
}

```
**元注解**
元注解（Meta-Annotation）就是用来修饰注解的注解，用来描述注解本身的行为。
Java 提供了 5 个元注解：
**① @Target——注解能贴在哪里**
```java
// @Target 决定了注解可以用在什么位置
@Target(ElementType.METHOD)  // 只能用在方法上
public @interface MyMethodAnnotation {
}
@Target(ElementType.TYPE)  // 只能用在类/接口上
public @interface MyClassAnnotation {
}
@Target({ElementType.METHOD, ElementType.FIELD})  // 可以用在方法和字段上
public @interface MyAnnotation {
}

//target可以的取值
public enum ElementType {
    TYPE,             // 类、接口、枚举
    FIELD,            // 字段（成员变量）
    METHOD,           // 方法
    PARAMETER,        // 方法参数
    CONSTRUCTOR,      // 构造方法
    LOCAL_VARIABLE,   // 局部变量
    ANNOTATION_TYPE,  // 注解类型（元注解用这个）
    PACKAGE,          // 包
    TYPE_PARAMETER,   // 类型参数（泛型）—— Java 8+
    TYPE_USE,         // 类型使用（任何使用类型的地方）—— Java 8+
    MODULE,           // 模块 —— Java 9+
    RECORD_COMPONENT  // Record 组件 —— Java 16+
}
```

**② @Retention——注解能活多久**
```java
// @Retention 决定了注解的生命周期
@Retention(RetentionPolicy.SOURCE)   // 只在源码中存在，编译后消失
@Retention(RetentionPolicy.CLASS)    // 保留到字节码中，但运行时获取不到
@Retention(RetentionPolicy.RUNTIME)  // 保留到运行时，可以通过反射获取

注解的生命周期：
═══════════════════════════════════════

            .java 源码    →    .class 字节码    →    JVM 运行时
            
SOURCE      ✅ 存在           ❌ 消失               ❌ 消失
            编译器可以看到     编译后被丢弃           

CLASS       ✅ 存在           ✅ 存在               ❌ 消失
            编译器可以看到     字节码中保留           运行时获取不到

RUNTIME     ✅ 存在           ✅ 存在               ✅ 存在
            编译器可以看到     字节码中保留           反射可以获取

```
**③ @Documented——是否出现在 JavaDoc 中**

**④ @Inherited——是否可以被继承**
**注意：`@Inherited` 只对类有效，对接口、方法、字段都无效。**
```java
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    
}

@MyAnnotation  // 父类加了注解
public class Parent {
    
}

// 子类没有加 @MyAnnotation，但因为 @Inherited 的存在
// 子类也"继承"了这个注解
public class Child extends Parent {
    // 通过反射获取 Child 的注解时，也能获取到 @MyAnnotation
}
```
**⑤ @Repeatable——（该注解）是否可以重复使用（Java 8+）**
jdk8 以前同一个对象不能贴两个一样的注解

### 2.注解的属性
**如何定义注解属性**
```java
// 定义一个有属性的注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestMapping {
    
    String path();              // 必填属性
    
    String method() default "GET";  // 有默认值的属性，可选
    
    String[] headers() default {};  // 数组类型的属性
}
```
**怎么给注解传参**
```java
// 使用注解时，给属性赋值 
@RequestMapping(path = "/users", (不填为默认值"GET")method = "POST") 
public void createUser() { 
}
```
**注解能取哪些属性**
```java
public @interface DemoAnnotation {
    
    // ✅ 基本数据类型
    int count() default 0;
    long timeout() default 1000L;
    boolean enabled() default true;
    double rate() default 0.5;
    
    // ✅ String
    String name() default "";
    
    // ✅ Class
    Class<?> targetClass() default Object.class;
    
    // ✅ 枚举
    RetentionPolicy policy() default RetentionPolicy.RUNTIME;
    
    // ✅ 注解（注解嵌套）
    Target target() default @Target(ElementType.METHOD);
    
    // ✅ 以上类型的数组
    String[] names() default {};
    int[] values() default {};
    
    // ❌ 不支持：包装类型（Integer、Long 等）
    // ❌ 不支持：集合类型（List、Map 等）
    // ❌ 不支持：自定义对象
}
```
**value 属性的特殊待遇**
如果注解只有一个属性，且名字叫 `value`，使用时可以省略属性名。
但如果有多个属性，就不能省略了。

### 3.注解是怎么起作用的
注解本身没有任何行为。它不会执行代码，不会修改逻辑，不会创建对象。真正让注解起作用的，是注解处理器（你自己写的一段程序，在运行时读取注解，并执行注解真正想做的逻辑）

**注解处理器怎么读取注解？**
反射！
```
反射获取注解的底层原理：
═══════════════════════════════════════

1. 你用 @interface 定义了注解
   → 编译器把它编译成一个接口

2. 你在代码上使用了注解（如 @MyTest）
   → 编译器把注解信息写入 .class 文件的属性表中

3. JVM 加载类时，读取 .class 文件
   → 如果注解的 @Retention 是 RUNTIME
   → JVM 会把注解信息保存在内存中

4. 你通过反射 API 获取注解
   → JVM 用动态代理创建一个注解接口的实现类
   → 这个实现类的方法返回你在代码中填写的属性值

```
**java提供的api**
```java
// 反射读取注解的核心方法（Class、Method、Field 都有这些方法）
//(class,method,field均能使用1，2，3)

// 1. 判断是否有某个注解
boolean hasAnnotation = method.isAnnotationPresent(MyAnnotation.class);

// 2. 获取某个特定注解
MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);

// 3. 获取所有注解
Annotation[] annotations = method.getAnnotations();

// 4. 获取注解的属性值:xx:属性名
//跟调用注解的方法一样。
//不过本质上就是调用了动态代理对象的方法
String value = annotation.xx();

```
**手写一个注解处理器**
```java
//自定义注解
import java.lang.annotation.*;

// 非空校验
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotNull {
    String message() default "字段不能为空";
}

// 长度校验
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Length {
    int min() default 0;
    int max() default Integer.MAX_VALUE;
    String message() default "字段长度不合法";
}

// 范围校验
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Range {
    int min() default 0;
    int max() default Integer.MAX_VALUE;
    String message() default "数值超出范围";
}


//自定义注解处理器
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

public class Validator {

    public static List<String> validate(Object obj) throws IllegalAccessException {
        List<String> errors = new ArrayList<>();
        Class<?> clazz = obj.getClass();
        
        for (Field field : clazz.getDeclaredFields()) {
            field.setAccessible(true);
            Object value = field.get(obj);
            
            // 检查 @NotNull
            if (field.isAnnotationPresent(NotNull.class)) {
                NotNull notNull = field.getAnnotation(NotNull.class);
                if (value == null) {
                    errors.add(notNull.message());
                }
            }
            
            // 检查 @Length
            if (field.isAnnotationPresent(Length.class)) {
                Length length = field.getAnnotation(Length.class);
                if (value != null && value instanceof String) {
                    String str = (String) value;
                    if (str.length() < length.min() || str.length() > length.max()) {
                        errors.add(length.message());
                    }
                }
            }
            
            // 检查 @Range
            if (field.isAnnotationPresent(Range.class)) {
                Range range = field.getAnnotation(Range.class);
                if (value instanceof Number) {
                    int num = ((Number) value).intValue();
                    if (num < range.min() || num > range.max()) {
                        errors.add(range.message());
                    }
                }
            }
        }
        
        return errors;
    }
}

//使用
public class ValidatorTest {

    public static void main(String[] args) throws Exception {
        
        // 测试1：正常数据
        User validUser = new User("张三", "zhangsan@example.com", 25);
        List<String> errors1 = Validator.validate(validUser);
        System.out.println("正常数据校验结果：" + (errors1.isEmpty() ? "通过" : errors1));
        
        // 测试2：用户名为空
        User nullNameUser = new User(null, "test@example.com", 25);
        List<String> errors2 = Validator.validate(nullNameUser);
        System.out.println("空用户名校验结果：" + errors2);
        
        // 测试3：用户名太短
        User shortNameUser = new User("A", "test@example.com", 25);
        List<String> errors3 = Validator.validate(shortNameUser);
        System.out.println("短用户名校验结果：" + errors3);
        
        // 测试4：年龄超出范围
        User oldUser = new User("李四", "lisi@example.com", 200);
        List<String> errors4 = Validator.validate(oldUser);
        System.out.println("年龄超范围校验结果：" + errors4);
    }
}
```

### 4.编译期注解处理——APT 机制
前面讲的反射，只能处理生命周期到运行时的注解（因为反射只能在运行时生效），
那对于编译期的注解呢？APT机制处理
经典例子：lombok
```
APT 的工作时机：
═══════════════════════════════════════

          编译期（javac 编译时）
               │
               ▼
    APT 介入，读取注解，生成新的 .java 文件
               │
               ▼
    生成的文件也会被编译
               │
               ▼
          最终的 .class 文件

和反射的区别：
- 反射：运行时处理注解，有性能开销
- APT：编译期处理注解，零运行时开销
————————————————
```

# 六.IO
## 1.File类
`java.io.File`类是文件和目录路径名的抽象表示，主要用于文件和目录的创建、查找和删除等操作。
具体方法见[java笔记](https://mcnerzykwkel.feishu.cn/wiki/BwO0wlcRwiuVfPkGMTocox1qnye)十八.1
## 2.IO流
分类
按数据流向：输入流，输出流
按数据类型：字节流，字符流
![[Pasted image 20260423213833.png]]
### 2.1字节流：（InputStream/OutputStream）
**什么时候用：** **二进制数据**（图片/音频/网络协议）用字节流；

#### 2.1.1字节输出流：OutputStream（抽象类）
`java.io.OutputStream`抽象类是字节输出流的所有类的超类，将指定的字节信息写出到目的地。它定义了字节输出流的基本共性功能方法。

**方法**：

| `public void close()`         | 先刷新缓冲区，再关闭此输出流并释放与此流相关联的任何系统资源 |
| ----------------------------- | ------------------------------ |
| `public void flush()`         | 刷新此输出流并强制任何缓冲的输出字节被写出          |
| public void write(`byte[] b`) | 将b写入到输出流                       |


##### 实现类：文件输出流FileOutputStream
`java.io.FileOutputStream`类是字节输出流OutputStream的一个子类，用于将数据写出到指定文件

#### 2.1.2字节输入流：InputStream（抽象类）
`java.io.InputStream`抽象类是表示字节输入流的所有类的超类，可以读取字节信息到内存中。它定义了字节输入流的基本共性功能方法。

**方法**

| `public void close()`        | 关闭此输入流并释放与此流相关联的任何系统资源       |
| ---------------------------- | ---------------------------- |
| `public abstract int read()` | 从输入流读取数据的下一个字节               |
| `public int read(byte[] b)`  | 从输入流中读取一些字节数，并将它们存储到字节数组 b 中 |
##### 实现类：文件输入流FileInputStream
`java.io.FileInputStream`类是字节输入流InputStream的一个子类，用于写入字节到指定文件中。

### 2.2字符流：
**特点**：
1. 输入流：一次读一个字节，遇到中文时，一次读多个字节
2. 输出流：底层会把数据按照指定编码进行编码，变成字节再写到文件中
**使用场景**：对于纯文本文件进行读写操作。

#### 2.2.1字符输入流：Reader（抽象类）
`java.io.Reader`抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。

**方法**：和字节输入流类似
##### 实现类：FileReader
**FileReader如何实现读取字符**
- 空参的read方法，一次获取一个字节，遇到中文一次读多个字节，把字节解码成十进制返回
- 带参的read方法，把读取字节、解码、强转三步合并了，强转之后的字符放到数组中

#### 2.2.2字符输出流Writer（抽象类）
`java.io.Writer`抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。

##### 实现类：FileReader
**FileReader何时输出
1. 情况一：缓冲区装满了
    1. 缓冲区刚好装满不会输出，再往缓冲流添加才会输出
2. 情况二：手动刷新，即flush
    1. `public void flush()`：清空缓冲区并把缓冲区的数据输出到文件中，此时还可以往文件中写出数据
3. 情况三：释放资源/关流，即close

### 2.3其他实现类
具体怎么用，上网查
####  2.3.1缓冲流
缓冲流，也叫高效流，是对4个基本的`FileXxx` 流的增强，所以也是4个流，按照数据类型分类：
- **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream`
- **字符缓冲流**：`BufferedReader`，`BufferedWriter`

缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

#### 2.3.2转换流
字符流增强：用于转换编码方式，例如将GBK编码的文本文件，转换为UTF-8编码的文本文件时用
字符输入流：InputStreamReader
字符输出流：OutputStreamWrite

#### 2.3.3序列化流和反序列化流
用于对象序列化和反序列化
字节输入流：`public ObjectInputStream(InputStream in)`：把基本流变成反序列化流。
字节输出流：`public ObjectOutputStream(OutputStream out)`： 把基本流包装成序列化流。

#### 2.3.4打印流
字节输出流：PrintStream
字符输出流：PrintWriter

#### 2.3.5解压缩流和压缩流
字节输入流：ZipInputStream
字节输出流：ZipOutputStream

**总结：怎么选实现类**
先根据读取类型确定是字节流还是字符流
再根据具体要求确定实现类。

## 3BIO,AIO,NIO
**区别**：
BIO（blocking IO）：就是传统的 java.io 包，它是基于流模型实现的，交互的方式是**同步、阻塞**方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。优点是代码比较简单、直观；缺点是 IO 的效率和扩展性很低，容易成为应用性能瓶颈。

NIO（non-blocking IO） ：Java 1.4 引入的 java.nio 包，提供了 Channel、Selector、Buffer 等新的抽象，可以构建**多路复用的、同步非阻塞 IO 程序**，同时提供了更接近操作系统底层高性能的数据操作方式。

AIO（Asynchronous IO） ：是 Java 1.7 之后引入的包，是 NIO 的升级版本，提供了**异步非堵塞**的 IO 操作方式，所以人们叫它 AIO（Asynchronous IO），异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。


# 七.序列化
参考：[序列化和反序列化](https://mp.weixin.qq.com/s/0EfIUB9E-0Oh_Clwuxswuw?poc_token=HNIX6mmjWOekFIDt9Sc3O9c0t4mIfHa9IzkMjHlL)
## 1.序列化是什么
序列化的原本意图是希望对一个Java对象作一下“变换”，变成字节序列，这样一来方便持久化存储到磁盘，避免程序运行结束后对象就从内存里消失，另外变换成字节序列也更便于网络运输和传播，所以概念上很好理解：
- **序列化**：把Java对象转换为字节序列。
- **反序列化**：把字节序列恢复为原先的Java对象。
而且序列化机制从某种意义上来说也弥补了平台化的一些差异，毕竟转换后的字节流可以在其他平台上进行反序列化来恢复对象。

## 2.如何序列化
例：
```java
//将要序列化的类
public class Student implements Serializable {  
  
    private String name;  
    private Integer age;  
    private Integer score;  
    // ... 其他省略 ...  
}
```
```java
//序列化
public static void serialize(  ) throws IOException {  
  
	  //创建将要序列化的对象
    Student student = new Student();  
    student.setName("CodeSheep");  
    student.setAge( 18 );  
    student.setScore( 1000 );  
  
  //用ObjectOutputStream序列化
    ObjectOutputStream objectOutputStream =   
        new ObjectOutputStream( new FileOutputStream( new File("student.txt") ) );  
    objectOutputStream.writeObject( student );  
    objectOutputStream.close();  
      
    System.out.println("序列化成功！已经生成student.txt文件");  
    System.out.println("==============================================");  
}
```
```java
//反序列化
public static void deserialize(  ) throws IOException, ClassNotFoundException {  
//用ObjectInputStream反序列化
    ObjectInputStream objectInputStream =   
        new ObjectInputStream( new FileInputStream( new File("student.txt") ) );  
    Student student = (Student) objectInputStream.readObject();  
    objectInputStream.close();  
      
    System.out.println("反序列化结果为：");  
    System.out.println( student );  
}
```

## 3.Serializable作用
上面在定义`Student`类时，实现了一个`Serializable`接口，然而当我们点进`Serializable`接口内部查看，发现它**竟然是一个空接口**，并没有包含任何方法！
![[Pasted image 20260424194350.png]]
观看`ObjectOutputStream`的`writeObject0()`源码发现
![[Pasted image 20260424194434.png]]
如果一个对象既不是**字符串**、**数组**、**枚举**，而且也没有实现`Serializable`接口的话，在序列化时就会抛出`NotSerializableException`异常！

原来`Serializable`接口也仅仅只是做一个标记用！！！

它告诉代码只要是实现了`Serializable`接口的类都是可以被序列化的！然而真正的序列化动作不需要靠它完成。

## 4. `serialVersionUID`作用
实验：先将上述student类序列化，然后再把student类稍加修改，之后再反序列化回去，发现报错，并且抛出了`InvalidClassException`异常：
![[Pasted image 20260424194654.png]]
这地方提示的信息非常明确了：序列化前后的`serialVersionUID`号码不兼容！

从这地方最起码可以得出**两个**重要信息：

- **1、serialVersionUID是序列化前后的唯一标识符**
- **2、默认如果没有人为显式定义过`serialVersionUID`，那编译器会为它自动声明一个！**
    
**第1个问题：** `serialVersionUID`序列化ID，可以看成是序列化和反序列化过程中的“暗号”，在反序列化时，JVM会把字节流中的序列号ID和被序列化类中的序列号ID做比对，只有两者一致，才能重新反序列化，否则就会报异常来终止反序列化的过程。

**第2个问题：** 如果在定义一个可序列化的类时，没有人为显式地给它定义一个`serialVersionUID`的话，则Java运行时环境会根据该类的各方面信息自动地为它生成一个默认的`serialVersionUID`，一旦像上面一样更改了类的结构或者信息，则类的`serialVersionUID`也会跟着变化！

## 5.两种特殊情况
### 5.1凡是被`static`修饰的字段是不会被序列化的
因为序列化保存的是**对象的状态**而非类的状态，所以会忽略`static`静态域也是理所应当的。

### 5.2凡是被`transient`修饰符修饰的字段也是不会被序列化的
**transient作用：**
如果在序列化某个类的对象时，就是不希望某个字段被序列化（比如这个字段存放的是隐私值，如：`密码`等），那这时就可以用`transient`修饰符来修饰该字段。
这样反序列化回来的对象，该字段的值为null

## 6序列化受控和增强
### 6.1序列化受控
**受控是什么意思：**
对象的属性值在我们的理想范围内

**为什么要受控：**
序列化和反序列化的过程其实是**有漏洞的**，因为从序列化到反序列化是有中间过程的，如果被别人拿到了中间字节流，然后加以伪造或者篡改，那反序列化出来的对象就会有一定风险了。

毕竟反序列化也相当于一种 **“隐式的”对象构造** ，因此我们希望在反序列化时，进行**受控的**对象反序列化动作。

**怎么控制：**
自行在将要序列化的类中编写`readObject()`函数，用于对象的反序列化构造，从而提供约束性。

**为什么`private`的`readObject()`能检测出反序列化对象受控**
`ObjectStreamClass`类的最底层源码：
![[Pasted image 20260424195511.png]]
答案是：反射

### 6.2单例模式增强
实验：将一个实现单例模式的类序列化，反序列化得到的对象与单例模式创造出来的对象不相同。
**解决办法是**：在单例类中手写`readResolve()`函数，直接返回单例对象，来规避之：


# 八.Java新特性
## 1.Lambda表达式
**是什么**
Lambda 表达式主要用于简化函数式接口（只有一个抽象方法的接口）的使用。

**怎么用**
其基本语法有以下两种形式：

(parameters) -> expression：当 Lambda 体只有一个表达式时使用，表达式的结果会作为返回值。
(parameters) -> { statements; }：当 Lambda 体包含多条语句时，需要使用大括号将语句括起来，若有返回值则需要使用 return 语句。

## 2.Stream流
### 1.Stream构建方法
```java
//单列集合
//Collection体系的集合可以使用默认方法stream()生成流
List<String> list = new ArrayList<String>();
Stream<String> listStream = list.stream();

//数组
//数组可以通过Arrays中的静态方法stream生成流
String[] strArray = {"hello","world","java"};
Stream<String> strArrayStream = Arrays.stream(strArray);
      
//同种数据类型
//同种数据类型的多个数据可以通过Stream接口的静态方法of(T... values)生成流
Stream<String> strArrayStream2 = Stream.of("hello", "world", "java");
Stream<Integer> intStream = Stream.of(10, 20, 30);

//双列集合：转换成keyset,valueset,entryset间接获取
Stream<String> keyStream = map.keySet().stream(); 
Stream<Integer> valueStream = map.values().stream(); 
Stream<Map.Entry<String, Integer>> entryStream = map.entrySet().stream();
```

### 2.中间操作方法
| 方法名                                                 | 说明                       |
| --------------------------------------------------- | ------------------------ |
| Stream`<T>` filter(Predicate predicate)             | 用于对流中的数据进行过滤             |
| Stream`<T>` limit(long maxSize)                     | 获取前maxSize个元素            |
| Stream`<T>` skip(long n)                            | 跳过前n个元素                  |
| Stream`<T>` distinct()                              | 元素去重，依赖HashCode和equals方法 |
| static `<T>` Stream`<T>` concat(Stream a, Stream b) | 合并a和b两个流为一个流             |
| Stream`<R>` map(Function`<T, R>` mapper)            | 转换流中的数据类型                |
```java
//Predivate<T>泛型接口是函数式接口，方法 public boolean test(T t)：对给定的参数进行判断，返回一个布尔值
list.stream().filter(s ->s.startsWith("张"))

// Stream`<R>` map(Function<T, R> mapper)：转换流中数据的数据类型 
//Function<T,R>泛型接口是函数式接口，T表示流中原本的数据类型，R表示要转成之后的类型 
//方法 public R apply(String T) 的形参T依次表示流里面的每一个数据，返回值R表示转换之后的数据
list.stream().map(new Function<String, Integer>() { 
	@Override
	 public Integer apply(String s) { 
	 ...
	return ...(Integer类型); } 
	});
//同样可以用lambda表达式简化
```

### 3.终结方法
| 方法名                                     | 说明            |
| --------------------------------------- | ------------- |
| void forEach(Consumer action)           | 遍历            |
| long count()                            | 统计流中的元素数      |
| Object[] toArray()                      | 收集流中的数据，放到数组中 |
| A[] toArray(IntFunction<A[]> generator) | 收集流中的数据，放到数组中 |
| R collect(Collector collector)          | 收集流中的数据，放到集合中 |

```java
//A[] toArray(IntFunction<A[]> generator)
//IntFunction`<R>`泛型接口是函数式接口，R表示具体类型的数组。
//方法：R apply(int value)中的value表示流中数据的个数，要跟数组长度一致，返回值为具体类型的数组，方法体创建数组
String[] arr2 = list.stream().toArray(value -> new String[value]);

```
R collect(Collector collector)
工具类Collectors提供了具体的收集方式：

|方法名|说明|
|---|---|
|public static `<T>` Collector toList()|把元素收集到List集合中|
|public static `<T>` Collector toSet()|把元素收集到Set集合中|
|public static Collector toMap(Function keyMapper,Function valueMapper)|把元素收集到Map集合中|
```java
//收集到list/set中
List<String> newList1 = list.stream().filter(s -> "男".equals(s.split("-")[1])).collect(Collectors.toList());

//收集到map中
//参数1表示键的生成规则，参数2表示值的生成规则
//function函数式接口，参数t为流中数据，返回值为键/值

Map<String, Integer> map2 = list.stream().collect(Collectors.toMap(s -> s.split("-")[0], s -> Integer.parseInt(s.split("-")[2])));
```

# 九.其他
## 1.== 和equals

**= =比较地址**
== 在 Java 里其实是个非常简单粗暴的比较方式。
- 如果你比较的是基本数据类型（比如 int、double），那它就直接比数值是不是相等
- 但如果你比较的是对象（引用类型），那 == 比较的就不是对象里面装的内容了，而是比较这两个变量是不是指向内存中同一个对象，也就是地址是否相同。
**equals比较内容**
equals 是 Object 类里定义的一个方法，所有的 Java 对象都继承了它。
- 它的默认行为其实和 == 一模一样，也是比地址。
- 但关键在于，很多常用的类（比如 String、Integer）都把这个方法给重写了。重写之后，equals 就不再比地址了，而是去比较对象里面实际存储的内容是否相等。

**hashcode和equals关系**
在 Java 中，对于重写 equals 方法的类，通常也需要重写 hashCode 方法，并且需要遵循以下规定：

一致性：如果a.equals(b),a.hashcode一定等于 b.hashcode
非一致性：如果a.hashcode== b.hashcode，a不一定equals b

重写 equals 方法时必须重写 hashCode 方法，以保证在使用哈希表等数据结构时，对象的相等性判断和存储查找操作能够正常工作。

## 2.String,StringBuilder,StringJoiner,StringBuffer

| 类型       | String   | StringBuilder | StringJoiner     | StringBuffer |
| -------- | -------- | ------------- | ---------------- | ------------ |
| **不可变性** | 不可变      | 可变            | 可变               | 可变           |
| 线程安全     | 是（因不可变）  | 否             | 否                | 是（同步方法）      |
| **性能**   | 低（频繁修改时） | 高（单线程）        | 高（单线程）           | 中（多线程安全）     |
| **适用场景** | 静态字符串    | 字符串拼接         | 字符串拼接（拼接处需插入符号时） | 多线程动态字符串     |
## 3.深拷贝浅拷贝
![[Pasted image 20260424203945.png]]
深拷贝和浅拷贝区别在复制对象的“深”与“浅”
  复制对象可能是调用对象.clone（clone方法继承自Object类，默认为浅拷贝），也可能指自己写的函数或其他方法
  对于非引用类型，深浅拷贝都能将原对象的指拷贝一份。
  但对于引用类型
- 浅拷贝：引用类型的值与原对象一致，也就是引用类型的地址值一致，也就是他们引用的其实是同一个对象。
- 深拷贝：而深拷贝是创建了一个新的对象，该对象内部属性与原对象所引用的对象的属性一致。递归的，新生成的对象深拷贝原对象所引用的对象。也就是如果原对象所引用的对象也有引用类型，新生成的对象也会生成一个属性一致的对象。

**如何实现深拷贝：**
1.实现 Cloneable 接口并重写 clone() 方法
2.序列化与反序列化
3.手动递归
