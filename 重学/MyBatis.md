# MyBatis执行器

![](.\pic\mybatis\执行器.jpg)

SqlSession中包含一个executor来实现真正的对数据库操作的功能，这个executor是CachingExecutor，构造函数传入BaseExecutor，实现二级缓存的功能，如果开启了二级缓存并且缓存存在，则返回缓存，否则调用传入的BaseExecutor相关的方法，BaseExecutor实现了一级缓存和获取连接的功能，首先执行query方法查看一级缓存中是否存在，存在则返回，否则执行子类的doQuery方法，从数据库中查询，查询到后放入一级缓存和二级缓存

- Executor提取了JDBC中一些共性，比如缓存、获取连接、事务等等

- ReuseExecutor

  编译过的sql语句可以重用，不需要再次编译，哪怕statementId不同，参数不同，只要sql语句相同就可以

# 一级缓存

一级缓存底层其实就是一个hashmap，sqlsession本身是线程不安全的，默认开启，只能修改作用范围，不能关闭

**一级缓存命中场景**

![](.\pic\mybatis\一级缓存命中.jpg)

手动清空缓存包括clearCache，commit，rollback，@Optional(flushCache=true)、@Update修饰的方法和update方法执行后会自动清除缓存，修改配置文件中的setting的localCacheScope=STATEMENT，是把一级缓存的作用域缩小为STATEMENT，也就是同一个查询语句的嵌套查询会用到一级缓存，而不是把一级缓存直接关闭了，只是每次查询完之后会清空缓存

# 二级缓存

**二级缓存扩展性需求**

![](.\pic\mybatis\二级缓存扩展性需求.png)

**二级缓存实现**

![](.\pic\mybatis\二级缓存实现方式.png)

定义了一个Cache接口，里面定义了缓存的存储、获取、大小等方法，针对每一个扩展功能，通过一个实现类来实现，再通过装饰者和责任链的方式进行功能组合，每个Cache中含有一个delegater，执行的过程中调用一个又一个的delegater

**二级缓存命中场景**

![](.\pic\mybatis\二级缓存命中场景.png)

- 二级缓存只有提交了之后（不包含自动提交）才能生效

- 同一个Mapper接口中使用@CacheNamespace或Xml中使用<cache/>并不会互通，接口和Xml都声明缓存空间会报错，缓存空间重复

  > 也就是说Mapper接口中使用了@CacheNamespace后，通过xml实现的方法并不会使用缓存，同理Xml中也是

- 可以通过引用缓存空间解决同一个Mapper和Xml之间的问题，也可以将多个缓存空间关联起来

**二级缓存执行流程**

![](.\pic\mybatis\二级缓存执行流程.png)

- 查询的时候是去二级缓存中查询，而填充和清空都需要等到提交之后才会更新二级缓存区，暂存区中只记录提交之后要对二级缓存做的修改，并不当作查询数据被查出

# Mybatis执行流程

## Mybatis自己的执行流程

![](.\pic\mybatis\Mybatis执行流程.png)

执行mapper的方法后，进入到mapper的动态代理类，使用反射交给SQLSession（DefaultSqlSession）来处理，SQLSession再交给Executor去执行

## 整合到Spring中后Mybatis的执行流程

![](.\pic\mybatis\Spring中Mybatis执行流程.png)

执行mapper的方法后，依然走Mybatis的Mapper的动态代理类，使用反射交给SqlSession（SqlSessionTemplate）来处理，SqlSessionTemplate也是一个实现了SqlSession接口的类，但是本身没有真正的处理功能，是交给内部的SqlSessionProxy（SqlSession）来处理的，SqlSessionProxy是一个动态代理类，在SqlSessionInterceptor类中方法里，才会从SqlSessionFactory真正的获取SqlSession来处理，如果开启了事务，将会把第一次获取的SqlSession放入ThreadLocal中，后面的执行语句依然从ThreadLocal中获取SqlSession，这样一级缓存或者事务的功能都能生效（事务和一级缓存是绑定线程的），如果没有开启事务，则每次会把存入ThreadLocal的SqlSession清空，导致后面每次的执行的语句都需要重新从SqlSessionFactory中再获取一次SqlSession

# StatementHandler

![](.\pic\mybatis\statementhandler.png)

声明Statement、填充参数、执行都是通过StatementHandler实现的，一次语句执行就会有一个StatementHandler来实现，由StatementHandler来判断是否要重用Statement

**StatementHandler实现类**

![](.\pic\mybatis\StatementHandler的实现类.png)

**StatementHandler执行流程**

![](.\pic\mybatis\StatementHandler执行流程.png)

## ParameterHandler

![](.\pic\mybatis\参数处理.png)

## ResultSetHandler

![](.\pic\mybatis\结果集处理.png)

# 谈谈你对Mybatis的理解

Mybatis是我们在工作中使用频率最高的一个ORM框架，持久层框架

1. 提供非常方便的API来实现增删改查操作
2. 支持灵活的缓存处理方案，一级缓存、二级缓存
3. 还支持相关的延迟数据加载的处理
4. 还提供了非常多的灵活标签来实现复杂的业务处理，if foreach where trim set bind
5. 相比于Hibernate会更加的灵活

# Mybatis的工作原理

Mybatis的基本情况：ORM框架

1. 系统启动时会加载mybatis的全局配置文件和对应映射文件，加载解析的相关信息存储在Configuration对象中，Configuration对象存储在SqlSessionFactory中
2. 通过SqlSessionBuilder对象build一个SqlSessionFactory对象（单例），建造者模式，（1）完成对配置文件的加载解析（2）完成SqlSessionFactory的创建
3. 通过SqlSessionFactory获取SqlSession对象，工厂模式
4. SqlSession有两种方式执行sql语句，（1）一种是直接调用selectList方法，传入statementId（命名空间+方法名）（2）一种是获取对应Mapper的代理对象，调用Mapper的方法，本质上还是执行了SqlSession的方法
5. 关闭会话（清空一级缓存）

通过Mybatis的执行流程来讲Mapper -》SqlSession -》 CacheExecutor -》 BaseExecutor -》具体的Executor -》StatementHandler -》 ParameterHandler -》 ResultSetHandler

# 介绍一下Mybatis中的缓存设计

1. 缓存的作用

   降低数据源的访问频率，从而提高数据源的处理能力，提高服务器的相应速度

2. Mybatis中的缓存设计

   - Cache接口，本地缓存，HashMap（一级缓存线程不安全，二级缓存通过Synchronized实现线程安全）
   - Mybatis中的缓存的架构设计：装饰者模式（二级缓存通过装饰者模式增强缓存的功能，如果关闭就没有CacheExecutor）
   - 一级缓存：session级别
   - 二级缓存：SqlSessionFactory级别
   - 二级缓存的命中率比一级缓存的命中率更高，所以先走二级缓存再走一级缓存

# 聊下Mybatis中如何实现缓存的扩展

1. 考察你的Mybatis中缓存架构的理解
2. 考察你对Mybatis缓存的扩展，实际动手能力
   - 创建Cache接口的实现，重写getObject和putObject方法
   - 怎么让我们自定义的实现：在cache标签中通过type属性关联我们自定义的Cache接口的实现

# Mybatis中涉及到的设计模式

- 缓存模块：装饰者模式
- 日志模块：适配器模式【策略模式】 代理模式
- 反射模块：工厂模式、装饰者模式
- Mapping：代理模式
- SqlSessionFactory：SqlSessionFactoryBuilder 建造者模式
- Executor：模版方法

# 谈谈你对SqlSessionFactory的理解

SqlSessionFactory是Mybatis中非常核心的一个API，是一个SqlSessionFactory工厂。目的是创建SqlSession对象，SqlSessionFactory应该是单例。SqlSessionFactory对象的创建是通过SqlSessionFactoryBuilder来实现，在SqlSessionFactoryBuilder即完成了SqlSessionFactory对象的创建，也完成了全局配置文件和相关的映射文件的加载和解析操作，相关的加载解析的信息会被保存在Configuration对象中

而且涉及到了两种设计模式：工厂模式、建造者模式

# 谈谈你对SqlSession的理解

SqlSession是Mybatis中非常核心的一个API：作用是通过相关API来实现对应的数据库数据的操作（内部是由Executor真正来处理）

SqlSession对象的获取需要通过SqlSessionFactory来实现。是一个会话级别的，当一个新的会话到来的时候。我们需要新建一个SqlSession对象来处理。当一个会话结束后我们需要关闭相关的会话资源，处理请求的方式：

1. 通过相关的增删改查的API直接处理
2. 可以通过getMapper(xxx.class)来获取相关的mapper接口的代理对象来处理

# 谈谈Mybatis中的分页原理

1. 谈谈分页的理解：数据太多，用户并不需要这么多，我们的内存也放不下这么多的数据

   ```
   SQL:
   	MYSQL:limit
   	Oracle:rowid
   ```

   

2. 谈谈Mybatis中的分页实现

   在Mybatis中实现分页有两种方式：

   - 逻辑分页：RowBounds 还是一次性查出所有数据，只是截取其中一部分作为结果返回
   - 物理分页：拦截器实现，PageHelper，拦截query方法在语句前后做动态拼接实现分页功能，mysql拼接limit

# Spring中是如何解决DefaultSqlSession的数据安全问题的

DefaultSqlSession是线程非安全的，也就意味着我们不能够把DefaultSqlSession声明在成员变量中，多个线程共享

在Spring中提供了一个SqlSessionTemplate来实现SqlSession的相关的定义（线程安全），然后在SqlSessionTemplate中的每个方法都通过SqlSessionProxy来操作，这是一个动态代理对象，每次执行的时候会在方法中获取SqlSession对象来实现相关的数据库的操作，这是方法级别的所以线程安全

# 谈谈你对Mybatis中的延迟加载的理解

延迟加载：等一会加载。在多表关联查询操作的时候可以使用到的一种方案，如果是单表操作就完全没有延迟加载的概念。比如，查询用户和部门信息，如果我们仅仅只是需要用户的信息，而不需要用户对应的部门信息，这时就可以使用延迟加载机制来处理

1. 需要开启延迟加载

   在配置中开启

2. 需要配置多表关联

   - association 一对一的关联配置
   - collection 一对多的关联配置

# 谈谈对Mybatis中插件的原理理解

Mybatis中的插件设计的目的是什么：方便我们开发人员实现对Mybatis功能的增强

设计中允许我们对：

- Executor
- ParameterHandler
- ResultSetHandler
- StatementHandler

这四个对象的相关方法实现增强

要实现自定义的拦截器：

1. 创建自定义的java类，通过@Interceptors注解来定义相关的方法签名，指定的类，方法，参数
2. 我们需要在对应的配置文件中通过plugins来注册自定义的拦截器

我们可以通过拦截器做哪些操作

1. 检查执行的SQL
2. 对执行的SQL的参数作处理
3. 对查询的结果做装饰处理
4. 对查询SQL的分表处理

# 使用Mybatis的的mapper接口调用时有哪些要求

Mybatis中的Mapper接口的实现的本质是代理模式

1. Mapper映射文件的namespace的值必须是Mapper接口对应的全类路径的名称
2. Mapper接口中的方法名必须在mapper的映射文件中有对应的sql的id
3. Mapper接口中的入参类型必须和mapper映射文件中的每个sql的parameterType类型相同
4. Mapper接口中的出参类型必须和mapper映射文件中的每个sql的resultType类型相同
5. 接口名称和Mapper映射文件同名

# 如何获取Mybatis中自增的主键

```xml
<insert id ="insert" useGenerateKeys="true" keyProperty="id"></insert>
```

```java
User user = new User();
userMapper.insert(user);
System.out.println(user.getId());
```

# 不同Mapper中的id是否可以相同

可以相同：id是通过namespace+id来判断是否唯一的，每一个映射文件的namespace都会设置为对应的mapper接口的全类路径名称，也就是保证了每一个Mapper映射文件的namespace是唯一的，那么我们只需要满足在同一个映射文件中的id是不同的就可以了

# 传统JDBC的不足和Mybatis的解决方案

1. 我们需要频繁的创建和释放数据库的连接对象，会造成系统资源的浪费，从而影响系统的性能。针对这种情况我们的解决方案是数据库连接池，然后在Mybatis中的全局配置文件中可以设置相关的数据库连接池，当然和Spring整合后我们也可以配置相关的数据库连接
2. SQL语句我们是直接添加到了代码中了，造成维护的成本增加。所以对应SQL的动态性要求比较高。这时我们可以考虑把SQL和我们的代码分离，在Mybatis中专门提供了映射文件。我们在映射文件中通过标签来写相关的SQL
3. 向SQL中传递参数也很麻烦，因为SQL语句的where条件不一定，可能有很多值也可能很少。占位符和参数需要一一对应，在Mybatis中自动完成java对象和SQL中参数的映射
4. 对于结果集的映射也很麻烦，主要是SQL本身的变化会导致解析的难度，我们的解决方案。在Mybatis中通过ResultSetHandler来自动把结果集映射到对应的java对象中
5. 传统的jdbc操作不支持事务，缓存，延迟加载等功能，在Mybatis中都提供了相关的实现

# Mybatis编程步骤是怎么样的

1. 创建SqlSessionFactory --》SqlSessionfactoryBuilder --》建造者模式 --》Configuration
2. 通过创建的SqlSessionFactory对象来获取SqlSession对象 --》Executor
3. 通过SqlSession对象执行数据库操作 --》API和Mapper接口代理对象 --》缓存 --》装饰者模式
4. 调用SqlSession中的commit方法来显示的提交事务 --》数据源和事务模块 --》JDBC和Managed
5. 调用SqlSession中的close方法来关闭回话

# 当实体中的属性和表中的字段不一致的情况下怎么办

1. 我们可以在对应的SQL语句中通过别名的方式来解决这个问题
2. 我们通过自定义resultMap标签来设置属性和字段的映射关系

# 如何设置Mybatis的Executor类型

1. 可以通过SqlSessionFactory的openSession方法中来指定对应的处理器类型
2. 可以通过全局配置文件中的settings来配置默认的执行器

# Mybatis中如何实现多个传参

1. 顺序传值

   ```java
   public void selectUser(String name, int deptId);
   ```

   ```xml
   <select id="selectUser" resultMap="baseResultMap">
   	select * from t_user where user_name = #{0} and dept_id = #{1};
   </select>
   ```

   #{}里面的数字代表的是入参的顺序

   但是这种方法不建议使用，SQL层次表达不直观，而且一旦顺序错了很难找到

2. @Param注解传值

   ```java
   public void selectUser(@Param("name")String name, @Param("deptId")int deptId);
   ```

   ```xml
   <select id="selectUser" resultMap="baseResultMap">
   	select * from t_user where user_name = #{name} and dept_id = #{deptId};
   </select>
   ```

   #{}里面的名称对应的就是@Param注解中修饰的名称

   这种方案推荐使用，直观

3. 通过Map传值

   ```java
   public void selectUser(Map<String, Object> map);
   ```

   ```xml
   <select id="selectUser" parameterType="java.util.Map" resultMap="baseResultMap">
   	select * from t_user where user_name = #{name} and dept_id = #{deptId};
   </select>
   ```

   #{}里面的名称就是Map中对应的Key

   这种方案适合传递多个参数，且参数灵活应变

4. 通过自定义对象传递

   ```java
   public void selectUser(User user);
   ```

   ```xml
   <select id="selectUser" parameterType="com.test.bean.User" resultMap="baseResultMap">
   	select * from t_user where user_name = #{name} and dept_id = #{deptId};
   </select>
   ```

   #{}中的名称就是自定义对象的属性名称

   这种方案也很直观，但是需要创建一个实体类，扩展不同意，需要添加属性，但是代码的可读性很高，业务逻辑处理也很方便

# 谈谈你对Mybatis中的日志的理解

1. Mybatis中的日志模块使用了适配模式 适配多种日志类型，有各种日志类型的专门的实现类
2. 如果我们需要适配Mybatis没有提供的日志框架，那么对应的需要添加相关的适配类，实现Log接口
3. 在全局配置文件中设置日志的实现 指定使用哪个类型的日志实现类，在解析配置文件的时候会覆盖掉LogFactory中默认的日志实现类
4. 在Mybatis的日志框架中提供了一个jdbc这个包，里面实现了JDBC相关操作的日志记录，使用代理对象在操作后添加日志打印

# 谈谈你对Mybatis中记录SQL日志的原理理解

在Mybatis中对执行JDBC操作的日志记录的本质是创建了相关核心对象的代理对象

- Connection -- ConnectionLogger
- PreparedStatement -- PreparedStatementLogger
- ResultSet -- ResultSetLogger

本质就是通过代理对象来实现的，代理对象中完成相关的日志操作，然后在调用对应的目标对象完成相关的数据库的操作处理

# 谈谈你对Mybatis中的数据源模块的设计理解

在Mybatis中单独设计了DataSource这个数据源模块，每个DataSource对应一个数据库，通过DataSourceFactory接口获取DataSource，DataSourceFactory有两种实现，一种非连接池UnpooledDataSource，一种连接池PooledDataSource，获取的DataSource也是非连接池DataSource和连接池DataSource。连接池DataSource内部包含非连接池DataSource，其实就是包装了一下非连接池DataSource，非连接池DataSource每次都是获取一个新的连接，

- 连接池DataSource是会看有没有空闲的连接，连接数是否到最大值，最老的连接是否已经超时，否则阻塞
- 数据库连接池关闭连接，如果空闲连接没有超过最大连接数那么久放回空闲队列中，否则关闭真实的连接

# 谈谈你对Mybatis中的事务模块的设计理解

1. 谈谈对事务的理解【ACID】

2. Mybatis中的事务管理

   ```xml
   <environments default="dev">
   	<environment id="dev">
       	<transactionManager type="JDBC"></transactionManager>
       </environment>
   </environments>
   ```

   - JDBC：在Mybatis中自己处理事务的管理，手动commit，rollback
   - Managed：在Mybatis中没有处理任何的事务操作，这种情况下的事务的处理会交给Spring容器来管理

# 谈谈你对Mapper接口的设计理解

1. 谈下Mybatis中Mapper接口对应的规则
2. 谈下Mybatis中的Mapper接口的设计原理 -- 代理模式的使用
3. 代理对象执行的逻辑的本质还是会执行SqlSession中相关的DML操作的方法
4. 为什么会多个代理对象（方便，根据接口获取动态代理类，直接调用方法）

# 谈谈你对Mybatis中的反射模块的理解

Reflector是Mybatis中提供的一个针对反射封装简化的模块：简化反射的相关操作。Mybatis是一个ORM框架，表结构的数据和java对中的数据的映射，不可避免的会存在非常多的反射操作

Reflector是一个独立的模块，里面的Reflector类对应一个java bean类，通过ReflectorFactory获取Reflector，通过ObjectFactory获取Object，MetaObject是封装了反射机制，可以简化对对象属性的操作，MetaClass是对复杂的属性解析的类，比如user.order.id

# 谈谈你对Mybatis的类型转换模块的理解

核心接口是TypeHandler，Mybatis中有很多通用的类型转换（BooleanTypeHandler、ByteTypeHandler。。。），作用就是将java的值转换为数据库的字段属性，或者是将数据库的字段属性转换为java的值

用户可以实现TypeHandler接口或者继承BaseTypeHandler来自定义一些类型转换，在Mybatis.xml中将自定义的TypeHandler注册进去即可

# 谈谈Mybatis和Spring整合的原理

- Spring和Mybatis的整合

  SqlSessionFactoryBean：一个FactoryBean，创建SqlSessionFactory，注入到Spring容器

  Mapper：代理对象，调用的SqlSessionTemplate的方法，最后调用真正的SqlSession，主要是为了实现事务功能，通过判断ThreadLocal中是否含有SqlSession来实现是否为同一事务

- SpringBoot项目中的整合

  AutoConfiguration：通过SqlSessionFactoryBean，获取到SqlSessionFactory注入到Spring容器中

# 谈谈你对Mybatis的整体理解

Mybatis是一个非常主流的半自动的ORM框架，非常简便的帮助我们完成相关的数据库操作

提供动态SQL，缓存和延迟加载等高级功能

然后整体的架构非常简单

- 外层接口
- 核心处理层
- 基础模块



































