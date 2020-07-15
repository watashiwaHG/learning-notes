## 编译
```
javac 文件名.java
```
## 解释or运行(没有.class后缀)
```
java 字节码文件
```
## 生成doc文档
```
javadoc 文件名.java
```
# 继承
子对象中包含一个父对象(并没有真正创建)

子类重写的成员变量并没有把父类的覆盖，直接使用成员变量的时候看引用类型。

子类的构造方法中会默认带一个super()，如果没有无参构造方法，就必须自己写上父类的构造方法。也就是说子类的构造方法中有一个父类的构造方法，并且是第一条语句。（所以不能多次调用）

子类的构造方法中可以使用this来调用其他的构造方法，此时的这个构造方法中不会有父类的构造方法，this和super在构造方法中不能共存。

只有子类构造方法才能调用父类构造方法。

![](./复习/继承内存图.png)

# 修饰符
final类不能被继承，方法不能被改写，所以和abstrac是违背的，不能一起使用。

final修饰的变量当做是常量处理。

定义navtive方法时，并不提供实现体，因为其实现体是用非Java语言在外面实现的。native可以和任何修饰符连用，abstract除外。因为native暗示这个方法时有实现体的，而abstract却显式指明了这个方法没有实现体。 



# 多态
子对象 父引用

只能访问属于父的成员（成员变量不能覆盖重写）

子方法如果重写访问子方法

子可以向上转型成父，之后还可以向下转型还原回来

instanceof判断当前对象是否为后面类型的实例

# 接口

接口中没有静态代码块和构造方法

接口之间可以多继承

特殊的抽象类（所有成员方法都为抽象pubic abstract）

在接口里面的变量默认都是public static final 的，它们是 ***公共的***,***静态的***,***最终的常量***, 相当于全局常量，可以直接省略修饰符。

实现类可以直接访问接口中的变量

Java7之前只有抽象方法，常量

Java8可以定义默认方法public default（为了不影响已经在使用的实现类）和静态方法public static

Java9可以定义私有方法(解决默认方法和静态方法中重复的代码，并且防止实现类调用)

当实现的多个接口有同名方法时，他们所有的方法信息也要相同。

如果实现类所实现的多个接口中，存在重复的默认方法，那么实现类必须对冲突的默认方法进行覆盖重写。

一个类如果直接父类当中的方法，和接口当中的默认方法产生了冲突，优先使用父类的方法。（继承优先于接口）

# 异常 throwable
1.	error 系统出错无法处理
2.	exception 可以处理
    * 。。。。。 必须处理 方法后throws 。。。。 方法里 try...catch 
    * runtime 可以不处理 <S>（猜测是因为已经帮我们处理了）</S>
# Integer

在-128到127的整数范围内

Integer.valueOf(20)和=20是返回相同的对象   ==比较的地址也是相同的

当Integer和int比较时，其实是把Integer拆包为int类型进行比较，所以最后比较的是值

jdk1.5之后自动装包(valueOf)、拆包(intValue)
# 八种基本类型
+ 整数型：
    + 字节型  byte   1字节
    + 短整型  short  2字节
    + 整型    int     4字节
    + 长整型  long  8字节
+ 浮点型：
    +  单精度浮点型  float  4字节
    +  双精度浮点型  double  8字节
+ 字符型：
    +   字符型  char  2字节
+ 布尔型：
    +  布尔型  boolean 1字节

byte、float、double没有缓存;

Integer、Short、Long的缓存范围都是[-128,127]; 

Character的缓存范围是[0,127]

# ASCII码表(0-127)Unicode(更全)

48 -> ‘0’

65 -> ’A’

97 -> ’a’

# 算术运算

表达式：用运算符连起来的式子叫做表达式

当两个不同类型的数据运算时，会转换成范围大的数据类型

byte short char运算时会自动提升为int型

自增自减
```java
int a = 2;
a++;
//正确写法 可以自增
byte b = 2;
b++;
//正确写法 可以自增
float c = 2.2F;
c++;
//正确写法 可以自增
String d = "123";
d++;
//错误写法 
char char1 = '1';
char1++;
System.out.println(char1);//结果为 2

char1 += 1;
System.out.println(char1);//结果为 3

System.out.println(char1 + 1);//结果为 52

char char2 = 'a';
char2++;
System.out.println(char2);//结果为 b

char2 += 1;
System.out.println(char2);//结果为 c

System.out.println(char2 + 1);//结果为 100

char char3 = 'a' + 1;
System.out.println(char3);//结果为b  

char char4 =(char) (char3 + 1);
System.out.println(char4);//结果为c

总结：只是表达式的时候会将char提升为int型，所以会变成对应的ASCII码
     而+= 或者 ++的时候会将结果的ASCII码也就是int型重新强制转换为char，所以最后的结果仍然是字符。
     a = a + 1的时候就需要自己在a+1的前面强制转换为char，否则编译会报错。
     常量运算赋值给数据类型为char，byte，short时，如果没有超过变量类型范围会自动转换为变量类型。原因是常量在javac编译的过程中会直接运算判断，在范围内自动强转，不在范围内再报错。但如果是含变量的运算的话，编译过程中并不知道变量具体的值，无法判断是否超出范围，所以也不能根据结果在范围内自动强转，只能直接报错。
```

赋值运算符
```java
byte a = 30;
a += 5;
//a = a + 5;
//byte = byte + int;
//byte = int + int;
//byte = int;
//byte = (byte) int; 
//赋值运算会强制转换
//a的数据类型依然为byte
//a = 35;
```

三元运算符
```java
int a = 3 > 4 ? 2.5 : 5; 
//错误写法 因为2.5为double类型不能直接赋值给int类型
System.out.println(3 > 4 ? 2.5 : 5);
//正确写法 不关心最后的类型
5 > 3 ? 4 : 2;
//错误写法 三元运算符得到的结果必须被使用
```

# switch
小括号内z还能是下列数据类型：
+ 基本数据类型: byte / short / int / char
+ 引用数据类型： String / enum枚举


# Java内存

Java内存分为五个区域：

1. **栈（Stack）** ：存放的都是方法的局部变量。**方法的运行一定在栈当中运行。**

2. **堆（Heap）** ：**凡是new出来的东西都在堆中。**

                堆内存里的东西都以一个地址值，16进制。
                堆内存里的数据都有默认值。规则：
                如果是整数，默认为0
                如果是浮点数，默认为0.0
                如果是字符，默认为'\u0000'
                如果是布尔类型，默认为false
                如果是引用类型，默认为null
3. **方法区（Method Area）** ：存储.class相关信息，包含方法的信息。

4. 本地方法栈（Native Method Stack）：与操作系统相关。

5. 寄存器（pc Register） ：与CPU相关。

![](./复习/java内存图.png)
![](./复习/java内存图2.png)
# 方法
传递参数时，方法内的参数其实是传入参数的副本，无论是基本类型的值，还是引用数据类型的引用，以前之所以会认为传入引用类型才会改变外部对象的值是因为确实改变了该引用指向的地址的对象的值，所以外部引用指向的地址的对象的值被改变了。而如果在方法内将引用的副本指向其他对象的话，就和基本类型相似了，只是改变了方法内部副本的值，而外部引用指向的对象没有发生变化，依然指向原来的对象。

方法可以和构造函数同名，构造函数是没有返回值的（连void都不是）



# static静态
![](./复习/static内存图.png)

# 泛型

\<E> 代表统一的类型，只能是引用类型（尖括号中可以任意名字）

泛型不存在继承关系，不确定时用?来接收

# Arrays

提供了大量static静态方法来操作数组[]
```java
public static String toString(数组) //将数组按照[元素1，元素2，元素3]转换为字符串
public static void sort(数组) //将数组的元素按照升序排列
/*
    如果是数值，按照从小到大进行排列
    如果是字符串，按照字母升序
    如果是自定义类，那么这个自定义类需要有Comparable和Comparator的接口支持
*/
```

# Math

提供了大量和数学有关的static静态方法

```java
public static *** abs(***)//取绝对值
public static *** ceil(***)//向上取整
public static *** floor(***)//向下取整
public static *** round(***)//四舍五入

public static final double PI //圆周率π值 
```

# final

final修饰的成员变量必须手动赋值，因为一旦给了默认值就不可变。可以直接赋值或者在构造方法中赋值。

# 权限修饰符

public(任意) > protect(子类) > (default)(同包) >private(本类)

只用在外部类和成员方法、成员变量上

外部类： public / (default)

成员内部类： public / protect / (default) / private

局部内部类：什么也不能写

# 匿名内部类

接口名称\父类名称 对象名称 = new 接口名称\父类名称(){
    覆盖重写抽象方法\成员方法
};

应用于只会使用一次的实现类或者子类



# 集合

简单记忆线程安全的集合类： 喂！SHE！  喂是指  vector，S是指 stack， H是指    hashtable，E是指：Eenumeration 

# String字符串

```java
//常用的构造方法
public String()
public String(byte[])
public String(char[])
String a = "123"

public boolean equals(Object)//区分大小写
public boolean equalsIgnoreCase(Object)//忽略大小写

public int length() //字符串长度
public String concat(String)//连接两个字符串，返回一个新的字符串
public int indexOf(String)//字符串第一次出现的索引，没有就返回-1
public char charAt(int)//获得索引对应的字符

public String substring(int begin)//返回索引从begin开始截取的字符串
public String substring(int begin,int end)//返回索引从begin开始到end结束的字符串 [begin,end)

public char[] toCharArray()//将当前字符串拆成字符数组作为返回值
public byte[] getBytes()//获得当前字符串底层的字节数组
public String replace(CharSequence oldStr,CharSequence newStr)//将字符串中的oldStr替换成newStr，返回新的字符串

public String[] split(String regex)//按照参数的规则，将字符串分成若干部分 参数其实是正则表达式 "."要换成"\\."
```
字符串的值永远无法改变 final char[] (java8) final byte[] (java9)

改变的只是引用所指向的对象，而不是对象的值

所有用=赋值和valueOf赋值的相同值的字符串其实指向的对象都是同一个

```java
//Java8所做的事情
String a = "123";
String b = "123";
System.out.println(a == b);//结果为true 指向同一对象
System.out.println(a.hashCode() + "  " + b.hashCode());//对象相同，hashCode必然相同
		
String c = String.valueOf("123");//与=创建的对象相同
System.out.println(c);
System.out.println(c == b);//结果为true 指向同一对象
System.out.println(c.hashCode());//对象相同，hashCode必然相同

String d = new String("123");//new方法必定新创建了一个对象 该对象拥有字符串"123"的值和hashCode
System.out.println(d == a);//结果为false 两个引用指向的对象不同
System.out.println(d.hashCode());//hashCode相同 因为在构造方法中将"123"这个字符串对象的hashCode复制给了d 所以hashCode相同对象不一定相同

String e = new String(a);//new方法必定新创建了一个对象 该对象拥有字符串"123"的值和hashCode
System.out.println(d == e);//结果为false 两个引用指向的对象不同
System.out.println(e.hashCode());//hashCode相同 因为在构造方法中将a这个字符串对象的hashCode复制给了e 所以hashCode相同对象不一定相同


String s1 = "a";
String s2 = "b";

String s3 = "ab";
String s4 = "a" + "b";
String s5 = s1 + "b";
String s6 = "a" + s2;
String s7 = s1 + s2;

s3 == s4 //true
s3 == s5 //false
s3 == s6 //false
s3 == s7 //false
//由变量加常量生成的字符串是在堆中新创建的（用final修饰也是常量）

String s8 = s5.intern();
s8 == s3 //true
//intern方法返回的字符串是在常量池中的
```
### String、StringBuffer和StringBuilder
String：不可更改，线程安全，效率低
StringBuffer：可更改，线程安全，效率高
StringBuilder：相当于没有synchronized的StringBuffer，可更改，线程不安全，效率更高。

StringBuffer（StringBuilder）初始容量为16，如果构造方法传入字符串，则为字符串长度+16，每次扩容原容量 * 2 + 2，如果还不够就为新添加字符串后的长度，因为扩容比较耗费资源，所以最好提前声明容量大小。
### Java9
![](./复习/String内存图.png)

# Date

```java
public Date()
//返回的是当前时间 CST代表中国标准时间
public Date(long)
//返回的是原点时间1970 1 1 8：00（东八区的原点时间）加上传入的毫秒所代表的时间

public long getTime()
//返回当前时间到原点时间的毫秒数
```

# SimpleDateFormat

是抽象类DateFormat的实现类

y 年 

M 月 

d 日

H 时

m 分

s 秒

```java
public SimpleDateFormat(String pattern)
//构造方法 按照传入的标准初始化
public Date parse(String)
//将传入的符合标准的字符串转化为Date
public String format(Date)
//将传入的日期转化为标准形式的字符串
```

# Calendar 日历类 操作日期

抽象类
```java
static Calendar getInstance()
//返回子类对象
public int get(int field)
//返回给定日历字段的值
public void set(int field,int value)
//将给定字段设定为给定值
public abstract void add(int field,int amount)
//将给定字段增加或减去相应的值
public Date getTime()
//返回一个表示此Calendar值的Date对象
```

# Collection集合

![](./复习/集合.png)
![](./复习/集合2.png)

### Collection中常用的方法
```java
public boolean add(E e)
//源码中只会返回true，否则可能就是报错
public void clear()
public boolean remove(E e)
public boolean contain(E e)
public boolean isEmpty()
public int size()
public Object[] toArray()
```
#Iterator接口
```java
public boolean hasNext()
public E next()
default E remove()
```

# 接口Comparable和Comparator

凡是实现了comparable接口的类都可以用Collections和Arrays的sort方法进行排序，自定义类也可以实现comparable接口的compareTo方法来实现sort排序

this - 传参 升序排序

comparator可以在使用sort(list,comparator)方法时用匿名实现的方式进行规定，效果和comparable一样
# ArrayList和LinkedList的区别
ArrayList和LinkedList都是不同步的，不能保证线程安全
ArrayList底层是Object数组,Linkedlist底层是双向链表数据结构
# ArrayList的扩容机制
以无参数构造方法创建ArrayList时，实际上初始化赋值的是一个空数组（在jdk1.7之前是初始一个容量为10的数组）。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10。
int newCapacity = oldCapacity + (oldCapacity >> 1),所以ArrayList每次扩容之后容量都会变为原来的1.5倍
如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE,否则，新容量大小则为_SIZE即为Integer.MAX_VALUE-8
# HashSet、LinkedHashSet、TreeSet
* HashSet：可以存null
* LinkedHashSet：HashSet的子类，对添加的元素增加两个引用指向上一次添加的元素和下一个添加的元素，所以可以按照添加的顺序遍历出来（当需要频繁的遍历时使用LinkedHashSet）
* TreeSet：底层红黑树（排序二叉树）
# HashTable和HashMap

#### HashTable
底层哈希表，单线程，线程安全,速度慢，不可以存储Null。早期类，基本被取代了，子类Properties是和IO流结合的集合，依然活跃，主要处理配置文件，key和value都是String类型。

#### HashMap
底层哈希表，多线程，线程不安全,速度快，可以存储Null（其他集合也差不多）
初始容量为16,每次扩容为之前的2倍
创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幕次方大小，也就是说HashMap总是使用2的幂作为哈希表的大小。
当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64时，此时此索引位置上的所有数据改为使用红黑树存储。
DEFAULT_INITIAL_CAPACITY :HashMap的默认容量，16
DEFAULT_LOAD_FACTOR: HashMap的默认加载因子0.75
threshold：扩容的临界值 = 容量 * 加载因子
TREEIFY_THRESHOLD: Bucket中链表长度大于该默认值，转化为红黑树8
MIN_TREEIFY_CAPACITY: 桶中的Node被树化时最小的hash表容量，64
# Collections工具类
可以将ArrayList和HashMap变为线程安全的

sychronizedXXX(XXX) 返回一个线程安全的
# 多线程

1. 实现方法
 
    * Thread类：创建一个类继承Thread类，重写run方法，调用start方法执行多线程。Thread的静态方法currentThread获得当前线程，在调用getName方法获得当前线程名称

    * Runnable接口：创建一个实现类，重写run方法，将对象传入Thread的构造方法，调用start方法执行多线程

    * Callable接口: jdk5.0之后新出现的，比起Runnable接口需要重写的是call方法，此方法拥有返回值（泛型），可以抛出异常，创建实现类对象后，需要传入FutureTask类的构造方法，再将FutureTask对象传入Thread的构造方法，调用Thread对象的start方法执行多线程，调用FutureTask对象的get方法可以获得返回值(需要等待多的线程执行结束)

    * Executors类（线程池实现工厂）：通过静态方法newFixedThreadPool(int 线程数量)获得ExecutorService（线程池）的实现类对象，再使用submit(Runnable)方法传入Runnable接口的实现类对象，实现多线程

2. 线程安全问题

    synchronized字段实现(Java内置，JVM实现，代码执行出现异常会自动释放资源)

    * synchronized(对象锁){
            代码块
        }
    * 访问修饰符 synchronized 返回值 方法名(参数列表){方法体}
    * 在以上方法的基础上使用对象锁的wait和notify方法保证线程安全

    Lock接口(代码层面，想保证锁被释放需要要写在finally里)

    * 调用实现类的lock和unlock方法
3. 线程通信(针对同一个监视对象的线程)
   wait() 进入阻塞状态，并且释放当前锁
   notify() 随机唤醒其他阻塞状态的线程
   notifyAll() 唤醒其他所有阻塞状态的线程
   必须使用在同步代码块和同步方法中，上述方法的调用者只能是同步代码块和同步方法中的同步监视器，否则出现异常
# 反射

![](./复习/反射.png)
![](./复习/Class类常用方法.png)
![](./复习/Class类成员常用方法.png)

# 网络通信协议
![](./复习/网络通信协议.png)
# SpringBoot配置ssl证书
server:
  port: 443
  ssl:
     key-store: classpath:3134781_watashiwahg.top.pfx
     key-store-type: PKCS12
     key-store-password: sJVt7gCx