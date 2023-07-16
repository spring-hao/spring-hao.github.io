---
title: Java基础
date: 2021-11-17 21:54:39.874
updated: 2022-08-12 17:49:19.043
url: /archives/java基础
categories: 
- java基础
tags: 
---

# 基本语法
## `++i`与`i++`
无论是`i++`和`++i`，对于 i 变量本身来说没有任何区别，执行的结果都是i变量的值加1,关键在于和=的结合
```java
        int i = 1;
        i++;
        ++i;
        System.out.println("i=" + i);//i=3
        int j = i++;
        System.out.println("j=" + j);//先将i赋值给j,再将i自增.i=4,j=3
        int m = ++i;
        System.out.println("m=" + m);//先将i自增,再将自增后的值赋给m,m=5,i=5
        System.out.println(m++);//先打印m后将m自增,打印结果是5,m=6
        System.out.println(++m);//m先自增后打印,打印结果是7
```

## ==和 equals 的区别
对于基本数据类型来说，==比较的是值。对于引用数据类型来说，==比较的是内存的地址。
> Java只有值传递，所以，对于==来说，不管是比较基本数据类型还是引用数据类型，其本质都是比较值，只是引用类型变量存的值是对象的地址。

**注意**：string类型重写了equals方法,比较的是值
`Object`类`equals()`方法：
```java
public boolean equals(Object obj) {
     return (this == obj);
}
```

```java
        String a = "aaa";
        String b = "aaa";
        String c = new String("aaa");
        System.out.println(a == b);	//true
        System.out.println(a == c);	//false
        System.out.println(a.equals(b));	//true
        System.out.println(a.equals(c));	//true
```
解析:
`String a = "aaa"`,内存会去查找永久代(常量池) ，如果没有的话，在永久代中中开辟一块儿内存空间，把地址付给栈指针，如果已经有了"aaa"的内存，直接把地址赋给栈指针。只在常量池中有一份内存空间，地址全部相同

只要是new String()，则，栈中的地址都是指向最新的new出来的堆中的地址

## hashCode()与 equals()
1. 如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。
> hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

2. HashSet 在对比的时候，同样的 hashcode 有多个对象，它会使用 equals() 来判断是否真的相同。也就是说 hashcode 只是用来缩小查找成本。
## comparable 和 Comparator 的区别
- `comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序 
- `comparator`接口实际上是出自`java.util`包它有一个`compare(Object obj1, Object obj2)`方法用来排序

需要对一个集合使用自定义排序时，就要重写`compareTo()`方法或`compare()`方法

## 赋值
JAVA的赋值运算是有返回值的，赋了什么值，就返回什么值
```java
Boolean flag = false;
if (flag = true)
{
    System.out.println("true");
}
else
{
    System.out.println("false");
}
```
输出true
## &&与&，||与|的区别
&&和&都是表示与，区别是&&只要第一个条件不满足，后面条件就不再判断。而&要对所有的条件都进行判断。
# int和Integer
**Java 语言虽然号称一切都是对象，但原始数据类型是例外**。8 种基本数据类型，分别为：

1. 6 种数字类型 ：byte、short、int、long、float、double，所占大小分别为1，2，4，8，4，8字节
2. 1 种字符类型：char，2字节
3. 1 种布尔型：boolean，1字节

> int是基本数据类型，默认值为0
> Integer是类，属于引用数据类型，默认值为null

Integer 是 int 对应的包装类，它有一个 int 类型的字段存储数据，并且提供了基本操作，比如数学运算、int 和字符串之间转换等。在 Java 5 中，引入了自动装箱和自动拆箱功能，Java 可以根据上下文自动进行转换，极大地简化了相关编程。**默认缓存是 -128 到 127 之间。**

## 理解自动装箱、拆箱
自动装箱实际上算是一种**语法糖**。 Java 替我们自动把装箱转换为 Integer.valueOf()，把拆箱替换为 Integer.intValue()，保证不同的写法在运行时等价，它们发生在**编译阶段**，也就是生成的字节码是一致的。

原则上，**建议避免无意中的装箱、拆箱行为**，尤其是在性能敏感的场合，创建 10 万个 Java 对象和 10 万个整数的开销可不是一个数量级的，不管是内存使用还是处理速度，光是对象头的空间占用就已经是数量级的差距了。
## 源码分析
缓存上限值实际是可以根据需要调整的，实现在 IntegerCache 的静态初始化块里。

```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];
        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            ...
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
        ...
  }
```

包装类里存储数值的成员变量“value”，不管是 Integer 还 Boolean 等，都被声明为“private final”，所以，它们同样是不可变类型！

## 原始类型线程安全
- 原始数据类型的变量，显然要使用并发相关手段，才能保证线程安全，如果有线程安全的计算需要，建议考虑使用类似 AtomicInteger、AtomicLong 这样的线程安全类。
- 特别的是，部分比较宽的数据类型，比如 float、double，甚至不能保证更新操作的原子性，可能出现程序读取到只更新了一半数据位的数值！

## Java 原始数据类型和引用类型局限性
- 原始数据类型和 Java 泛型并不能配合使用
这是因为 Java 的泛型某种程度上可以算作伪泛型，它完全是一种**编译期的技巧**，Java 编译期会自动将类型转换为对应的特定类型，这就决定了使用泛型，必须保证相应类型可以转换为 Object。

- 无法高效地表达数据，也不便于表达复杂的数据结构，比如 vector 和 tuple
Java 的对象都是引用类型，如果是一个原始数据类型数组，它在内存里是一段连续的内存，而对象数组则不然，数据存储的是引用，对象往往是分散地存储在堆的不同位置。这种设计虽然带来了极大灵活性，但是也导致了数据操作的低效，尤其是无法充分利用现代 CPU 缓存机制。

# String, StringBuffer, StringBuilder
String 被声明为 final，因此它不可被继承，不可变，线程安全。也由于它的不可变性，类似拼接、裁剪字符串等动作，都会产生新的 String 对象。
`StringBuilder`与`StringBuffer`都继承自`AbstractStringBuilder`类，在`AbstractStringBuilder`中也是使用字符数组保存字符串`char[]value`，但是没有用`final`关键字修饰，所以这两种对象都是可变的。StringBuffer 本质是一个线程安全的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销

## 1. 可变性
- String 不可变
- StringBuffer 和 StringBuilder 可变
## 2. 线程安全
- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步
## 3. 字符串缓存
intern() 方法，目的是提示 JVM 把相应字符串缓存起来，以备重复使用。在创建字符串对象并调用 intern() 方法的时候，如果已经有缓存的字符串，就会返回缓存里的实例，否则将其缓存起来。Intern 是一种**显式地排重机制**，但是它也有一定的副作用，因为需要开发者写代码时明确调用，一是不方便，每一个都显式调用是非常麻烦的；另外就是我们很难保证效率，应用开发阶段很难清楚地预计字符串的重复情况

```java
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;//new StringBuilder().append("a").append("b").toString()    new String("ab")
        String s5 = "a" + "b";
        String intern = s4.intern();//将这个字符串对象尝试放入串池,如果有则不放入,如果没有就放入并返回串池中的对象
        System.out.println(s3 == s4);//false
        System.out.println(s3 == s5);//true
        System.out.println(s3 == intern);//true
```
## 使用的总结：
1. 操作少量的数据: 适用 String
2. 单线程操作字符串缓冲区下操作大量数据: 适用 StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据: 适用 StringBuffer

# 关键字
## instance 关键字
instance是java的二元运算符，用来判断他左边的对象是否为右面类（接口，抽象类，父类）的实例
## final 关键字
最终的、不可修改的，用来修饰**类、方法和变量**，具有以下特点：
1. final 修饰的类不能被继承，final 类中的所有成员方法都会被隐式的指定为 final 方法；
2. final 修饰的方法不能被重写；
3. final 修饰的变量是常量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能让其指向另一个对象。

## this 关键字
this关键字只能在方法内部使用，表示对“调用方法的那个对象”的引用，this引用会自动应用于同一个类中的其他方法，只有当需要明确指出对当前对象的引用时才需要使用this关键字。

`return this`直接返回当前对象的引用，常常用于链式操作。

在构造器中如果为this添加了参数列表，那么将产生对符合此参数列表的某个构造器的明确调用。尽管可以用this调用一个构造器，但却不能调用两个，除此之外，必须将构造器调用置于最起始处，否则会编译报错。除构造器之外，编译器禁止在其他任何方法中调用构造器

## static 关键字
1. 修饰**成员变量和成员方法**，不能修饰接口，接口只能用public和abstract修饰
2. 静态代码块: 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块**只执行一次**.
3. 静态内部类（static 修饰类的话只能修饰内部类）： 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：a. 它的创建是不需要依赖外围类的创建。b. 它不能使用任何外围类的非 static 成员变量和方法。

## super 关键字
用于从子类访问父类的变量和方法
**注意:**
- 构造器中使用`super()`调用父类中的其他构造方法时，该语句必须处于构造器的首行，this 调用本类中的其他构造方法时，也要放在首行
- this、super 不能用在 static 方法中
## 访问修饰符
|访问修饰符|访问范围|继承性|
|-------|-------|-------|
|private |本类内部|不可继承|
|default |本类+同包|同包子类可以继承|
|protected |本类+同包+子类|可以继承|
|public |公开|可以继承|

# 方法
## 泛型
在没有泛型类之前，必须使用Object编写适用于多种类型的代码，这中操作繁琐且不安全。在Java中由于继承和向上转型，子类可以非常自然地转换成父类，但是会丢失子类特有的方法，若子类重写父类的方法则不会丢失。而除非确切知道所要处理的对象的类型，否则向下转型几乎是不安全的，如果向下转型为错误的类型，就会得到一个运行时错误的异常。

泛型程序设计意味着编写的代码可以对多种不同类型的对象重用。在进行泛型操作时，编译器会检查传入的参数是否为指定泛型，这比传一个Object类型的参数要安全得多。出现编译错误要比运行时出现类的强制类型转换异常好得多，泛型使程序更易读、更安全。
## 方法的类型
1. 无参数无返回值的方法
2. 有参数无返回值的方法
3. 有返回值无参数的方法
4. 有返回值有参数的方法
5. return 在无返回值方法的特殊使用
```java
public void f5(int a) {
    if (a > 10) {
        return;//表示结束所在方法 （f5方法）的执行,下方的输出语句不会执行
    }
    System.out.println(a);
}
```
## 静态方法和实例方法
静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员（即实例成员变量和实例方法）`类名.方法名`，而实例方法不存在这个限制。

## 重载和重写
重载是方法根据传入参数名字的不同，自动选择不同的方法执行(编译时就确定)
重写的本质是根据方法接收者的实际类型来选择方法版本
> 深入理解Java虚拟机 P311

### 重载
发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法**返回值和访问修饰符**可以不同
> 不能有两个名字相同，参数相同，返回值或修饰符不同的方法
```java
    void a(int a) {
        return ;
    }
//错误
    int a(int a) {
        return 1;
    }
```
**静态分派与重载**：
```java
static class Man extends Human{}
static class Woman extends Human{}
//静态类型Human   实际类型Man
Human human = (new Random()).nextBoolean() ? new Man() : new Woman();//父类指向子类对象,多态
```
- 静态类型：静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型在编译期可知
- 实际类型：实际类型变化的结果在运行时才确定，编译器在编译时不知道一个对象的实际类型是什么
> 代码中对象human的实际类型在编译器是一个“薛定谔的人”，必须等到程序运行到这行代码才能确定。而human的静态类型编译时就知道了，也可以在使用时强制类型转换来改变这个类型，但这个改变在编译器仍然可知。

虚拟机在重载时是通过参数的静态类型而不是实际类型作为判断依据的，由于静态类型在编译器已知，所以在编译期间就决定了使用哪个重载版本。

所有依赖静态类型来决定方法执行版本的分派动作都成为静态分派，静态分派最典型的表现就是方法重载，静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机执行的。

### 重写
子类对父类的**允许访问**的方法的实现过程进行重新编写。
1. 返回值类型、方法名、参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。
2. 如果父类方法访问修饰符为 private/final/static 则子类就不能重写该方法，但是被 static 修饰的方法能够被再次声明
3. 构造方法无法被重写

## 构造方法
一个类即使没有声明构造方法也会有默认的不带参数的构造方法。如果自己添加了类的构造方法（无论是否有参），Java 就不会再添加默认的无参数的构造方法了
### 特点
1. 名字与类名相同
2. **没有返回值**，但是不能用void声明构造函数
3. 生成对象时自动执行


构造方法不能被 override（重写），但是可以 overload（重载）


# 异常
- error：属于程序无法处理的错误，没办法通过 catch 来进行捕获，大多数错误与代码编写者所执行的操作无关
- 检查性异常：最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。代码在编译过程中，如果受检查异常没有被 catch/throw 处理的话，就没办法通过编译
- 运行时异常：运行时异常程序员导致的异常。即使不处理此类异常也可以正常通过编译，并不强制进行显示处理

## 异常的结构
![Java异常类层次结构图.png](/upload/2021/10/Java%E5%BC%82%E5%B8%B8%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E5%9B%BE-c444dd42d5db472bb4c305f9281e7b15.png)

`RuntimeException`及其子类都统称为非受检性异常，例如：`NullPointerException`、`NumberFormatException`（字符串转换为数字）、`ArrayIndexOutOfBoundsException`（数组越界）、`ClassCastException`（类型转换错误）、`ArithmeticException`（算术错误）等

## try-catch-finally

- try块： 用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。
- catch块： 用于处理 try 捕获到的异常。若有一个catch语句匹配到了，则执行该catch块中的异常处理代码，就不再尝试匹配别的catch块了。
- finally 块： 无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。

当 try 语句和 finally 语句中都有 return 语句时，在方法返回之前，finally 语句的内容将被执行，并且 finally 语句的返回值将会覆盖原始的返回值。如下：
```java
public class Test {
    public static int f(int value) {
        try {
            return value * value;
        } finally {
            if (value == 2) {
                return 0;
            }
        }
    }
}
//如果调用 f(2)，返回值将是 0，因为 finally 语句的返回值覆盖了 try 语句块的返回值
```
# 易错
## 1. 运算符关系

- 赋值=，最后计算
- =右边的从左到又依次压入**操作数栈**
- 实际计算过程看运算符优先级
- 自增，自减操作都是直接修改变量值，不经过**操作数栈**
- 临时结果也是存储在**操作数栈**中

|优先级	|运算符	|结合性|
|-------|-------|-------|
|1	|()、[]、{}	|从左向右|
|2	|!、+、-、~、++、-\-	|从右向左|
|3	|*、/、%	|从左向右|
|4	|+、-	|从左向右
|5	|«、»、>>>	|从左向右|
|6	|<、<=、>、>=、instanceof	|从左向右|
|7	|==、!=	|从左向右|
|8	|&	|从左向右|
|9	|^	|从左向右|
|10	|\|	|从左向右|
|11	|&&	|从左向右|
|12	|\|\|	|从左向右|
|13	|?:	|从右向左|
|14	|=、+=、-=、*=、/=、&=、\|=、^=、~=、«=、»=、>>>=	|从右向左|


```java
        int x = 0,y = 1;

        /*

        if (++x == y-- & x++ ==1 || --y == 0)
        {
            System.out.println("x="+x+",y="+y);      //x = 2,y = 0;
        }
        else
        {
            System.out.println("y="+y+",x="+x);
        }
        */
        

        if(++x == y--)
            System.out.println("y="+y+",x="+x); //成立！  //y=0 x=1
        else
            System.out.println("x="+x+",y="+y);
```
其中：
`++x`先计算再其他，`x++`先其他再计算，输出为计算后的结果！

第二个if语句输出：先计算`++x`为1，再判断是否与y相等（y判断后`--`）；结果y计算了一遍，输出了`y--`为0

`||`左边成立不再计算右边；`|`即使左边成立也要计算右边！

## 2. 代码注释问题
Java中注释不会被编译，注释量不影响编译后的程序大小

## 3. Java单例模式(线程安全)
单例模式要点: 
1. 只能有一个实例
	- 构造器私有化
2. 必须自行创建这个实例
	- 含有一个该类的静态变量来保存这个唯一的实例
3. 必须自行向整个系统提供这个实例
	- 对外提供获取该实例对象的方式

**静态内部类方式**

不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。
静态内部类有着一个致命的缺点，就是传参的问题，由于是静态内部类的形式去创建单例的，故外部无法传递参数进去
```java
public class SingleObject {

   //静态内部类（不会随着外部类的初始化而初始化）创建 SingleObject 的一个对象
   private static class Inner{
	private static SingleObject instance = new SingleObject();
}

   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}
 
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return Inner.instance;
   }
}
```

**双重校验锁实现对象单例（线程安全）**
```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```
`uniqueInstance`采用`volatile`关键字修饰也是很有必要的，`uniqueInstance = new Singleton()`; 这段代码分三步执行： 
1. 为`uniqueInstance`分配内存空间 
2. 初始化`uniqueInstance`
3. 将`uniqueInstance`指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用`getUniqueInstance()`后发现`uniqueInstance`不为空，因此返回`uniqueInstance`，但此时`uniqueInstance`还未被初始化。

## 4. try-catch-finally-普通
首先进入try代码块，若不抛出异常，则catch不运行，finally运行，接着继续往下运行普通代码；若抛出异常，则运行catch，由于catch中有return语句，finally中会在catch中的return语句之前运行

finally是在return后面的表达式运算后执行的（此时并没有返回运算后的值，而是先把要返回的值保存起来，管finally中的代码怎么样，返回的值都不会改变，任然是之前保存的值），所以函数返回值是在finally执行前确定的

```java
    public static String sRet = "";
    public static void func(int i)
    {
        try
        {
            if (i%2==0)
            {
                throw new Exception();
            }
        }
        catch (Exception e)
        {
            sRet += "0";
            return;
        }
        finally
        {
            sRet += "1";
        }
        sRet += "2";
    }
    public static void main(String[] args)
    {
        func(1);	//12
//        func(2);	//01
        System.out.println(sRet);
    }
```

## 5. 类的加载顺序
1. 父类静态对象和静态代码块
2. 子类静态对象和静态代码块
3. 父类非静态对象和非静态代码块
4. 父类构造函数
5. 子类非静态对象和非静态代码块
6. 子类构造函数

其中：类中静态区域按照声明顺序执行，并且(1)和(2)不需要调用new类实例的时候就执行了(在类加载到方法区的时候执行)

静态块：用staitc声明，jvm加载类时执行，仅执行一次
构造代码块：类中直接用{}定义，每一次创建对象时执行
**执行顺序优先级：静态域,main(),构造代码块,构造方法。**

## 6. 重写原则
方法名相同，参数类型相同，子类中可能需要调用父类方法，因此需要满足两同两小一大原则：
- 方法名相同，参数类型相同
- 子类返回类型小于等于父类方法返回类型
- 子类抛出异常小于等于父类方法抛出异常
- 子类访问权限大于等于父类方法访问权限

## 7. 类型转换
(byte1，short2，char2)--int4--long8--float4--double8
按照字节数由高到低
小数如果不加 f 后缀，默认是double类型。

## 8. 成员变量与局部变量
- 就近原则
- 变量的分类
	- 成员变量:类变量,实例变量
	- 局部变量
- 非静态代码块:每次创建实例都会执行
- 方法调用:调用一次执行一次

```java
public class Test {
    static int s;
    int i;
    int j;

    {
        int i = 1;
        i++;
        j++;
        s++;
    }

    public void test(int j) {
        j++;
        i++;
        s++;
    }

    public static void main(String[] args) {
        Test test1 = new Test();
        Test test2 = new Test();
        test1.test(10);
        test1.test(20);
        test2.test(30);
        System.out.println(test1.i + "," + test1.j + "," + test1.s);//2,1,5
        System.out.println(test2.i + "," + test2.j + "," + test2.s);//1,1,5
    }
}
```
踩坑点:
1. 用`static`修饰的所有类共享,不管用哪个对象,指向的都是同一数据
2. 代码块中定义的变量同样有作用域
3. `int`类型的默认值是0
4. 就近原则可以被`this`关键字打破

## 9. 抽象类和接口
接口是对行为的抽象，它是抽象方法的集合，利用接口可以达到 API 定义和实现分离的目的。接口，不能实例化；不能包含任何非常量成员，任何 field 都是隐含着 public static final 的意义

抽象类是不能实例化的类，用 abstract 关键字修饰 class，其目的主要是代码重用。除了不能实例化，形式上和一般的 Java 类并没有太大区别，可以有一个或者多个抽象方法，也可以没有抽象方法。抽象类大多用于抽取相关 Java 类的共用方法实现或者是共同成员变量，然后通过继承的方式达到代码复用的目的。

**Java 不支持多继承，**为接口添加任何抽象方法，相应的所有实现了这个接口的类，也必须实现新增方法，否则会出现编译错误。对于抽象类，如果我们添加非抽象方法，其子类只会享受到能力扩展，而不用担心编译出问题。

接口的职责也不仅仅限于抽象方法的集合，其实有各种不同的实践。有一类没有任何方法的接口，通常叫作 Marker Interface，顾名思义，它的目的就是为了声明某些东西，比如熟知的 Cloneable、Serializable 等。

Java 8 增加了函数式编程的支持，所以又增加了一类定义，即所谓 functional interface，简单说就是只有一个抽象方法的接口，通常建议使用 @FunctionalInterface Annotation 来标记。Lambda 表达式本身可以看作是一类 functional interface
**语法层面上的区别：**
1. 抽象类可以提供成员**方法的实现**细节，而接口中方法无实现；
2. 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是public static final类型的；
3. 接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法；
4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。
5. 抽象类不允许被实例化，只能被继承。

**设计层面上的区别：**
1. 抽象类是对成员变量和方法的抽象，是一种 is-a 关系，是为了解决代码复用问题。接口仅 仅是对方法的抽象，是一种 has-a 关系，表示具有某一组行为特性，是为了解决解耦问 题，隔离接口和具体的实现，提高代码的扩展性。
2. 设计层面不同，抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计。也就是说对于抽象类，如果需要添加新的方法，可以直接在抽象类中添加具体的实现，子类可以不进行变更；而对于接口则不行，如果接口进行了变更，则所有实现这个接口的类都必须进行相应的改动。

如果要表示一种 is-a 的关系，并且是为了解决代码复用问题， 那么就用抽象类;如果要表示一种 has-a 关系，并且是为了解决抽象而非代码复用问题，那就用接口。

## 10. finalize
它是object中的一个方法，若子类重写它，垃圾回收是就会调用此方法，不过将一些资源释放操作或清理操作放在finalize方法中非常不好，严重影响性能，甚至可能会导致OOM，从Java9开始已经被标记为废弃不建议使用了。
- 当重写了finalize方法的对象，在构造方法调用时，JVM会将其包装成Finalizer对象并将其加入到unfinalize队列中。 
- 不同对象的finalize方法调用顺序并没有保证
- finalize方法中若出现异常，不会进行输出
- 重写了finalize方法的对象在第一次被gc时，并不能及时释放它所占用的内存，要等守护线程执行完finalize方法并把它从unfinalize队列移除后，第二次gc时才能彻底移除。这就导致不能及时释放内存，增加出现OOM的错误。 