# 为什么不建议直接使用@Async

@Async是Spring框架中提供的一个可以标注一个方法或一个类中的方法为异步执行的注解

> Spring会通过SimpleAsyncTaskExecutor线程池获取线程执行@Async的方法，这个线程池不会重用线程，并且每次调用都会创建一个新的线程，也没有最大线程数的设置，所以在并发量大的时候会产生严重的性能问题
>
> 一般生产环境，我们建议使用自定义线程池配合@Async注解一起使用，而不应该直接使用@Async注解

# @Lazy注解

1. 延迟创建

   非懒加载的单例bean在Spring容器启动时，就会完成创建；而使用了@Lazy注解修饰的懒加载的单例bean，只有当首次被使用的时候才会去创建。这类bean一般被如下注解组合修饰：@Component+@Lazy；@Bean+@Lazy；@Configuration+@Lazy等

2. 延迟注入

   当@Lazy注解和@Autowired注解搭配使用，或者应用在构造方法之上，或者应用在构造方法的属性之上时，Spring都会先给该属性注入一个代理对象，只有在首次访问该属性时，Spring才会执行该代理对象的逻辑，给该属性注入一个真正的bean对象

# Spring中AOP失效的场景

AOP是由java的动态代理实现的，也就是说java的动态代理不能工作的场景，aop就会失效

1. 当前类没有被Spring容器所管理，Spring的AOP是在Bean创建的初始化后阶段进行的，如果当前类没有被Spring容器所管理，那么他的Spring AOP功能肯定会失效

2. 同一个类中方法的调用

   ```java
   class A {
       public void a() {
           b();
       }
       public void b() {
           // ....
       }
   }
   ```

   > 注入当前类，就能拿到当前类的代理对象

3. 内部类方法的调用，该方法会直接调用内部类实例对象的方法，同样没有使用代理对象，所以AOP会失效

4. 私有方法，代理对象是无法调用的

5. static修饰的方法，因为static修饰的方法属于类对象，而不属于对象实例，所以无法被代理对象调用

6. final修饰的方法，因为被final修饰的方法是无法被重写的，所以代理对象也是无法调用的

# Spring中事务失效的场景

1. 当前类没有被Spring容器所管理
2. 同一个类中方法的调用，导致AOP失效，从而导致@Transaction注解失效
3. @Transactional注解应用在非public修饰的方法上，在Spring的源码中，要求我们所定义的事务方法必须是public修饰的，否则Spring的事务就会失效
4. @Transactional注解在了static修饰的方法上
5. @Transactional注解在了final修饰的方法上
6. 多个事务方法不在同一个线程内执行，不同线程获取到的数据库连接是不同的，不同的数据库连接会导致事务失效
7. 数据库引擎不支持事务
8. @Transactional注解的propagation事务传播行为属性设置不当，如果将事务方法的事务传播行为设置为了不支持事务的传播类型，那么该事务方法的事务将会失效
9. @Transactional注解的rollbackFor属性设置不当，导致回滚的异常和抛出的异常不匹配，事务不会回滚
10. 事务方法内异常被捕获而导致的事务失效