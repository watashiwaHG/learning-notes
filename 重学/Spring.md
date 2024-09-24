# 谈谈对Spring的理解

java框架，轻量级，全家桶

AOP IOC

# 谈谈你对Spring IOC的理解

**总**：

控制反转：理论思想，原来得对象是由使用者自己来控制，有了spring之后，可以把整个对象交给spring来帮我们进行管理

DI：依赖注入，把对应的属性 的值注入到具体的对象中，@Autowired，populateBean完成属性值的注入

容器：存储对象，使用map结构来存储，在spring中存在三级缓存，singletonObjects存放完整的bean对象，整个bean的生命周期，从创建到使用到销毁的过程全部都是由容器来管理（bean的生命周期）

**分**：

1. 一般聊ioc容器的时候要涉及到容器的创建过程（beanFactory，DefaultListableBeanFactory，ApplicationContext），向bean工厂中设置一些参数（BeanPostProcessor，Aware接口的子类）等等属性
2. 加载解析bean的对象，准备要创建的bean对象的定义对象beanDefinition（xml或者注解的解析过程）
3. beanFactoryPostProcessor的处理，此处是扩展点（PlaceHolderConfigureSupport，ConfigurationClassPostProcessor）
4. BeanPostProcessor的注册功能，方便后续对bean对象完成具体的扩展功能
5. 通过反射的方式将BeanDefinition对象实例化为具体的bean对象
6. bean对象的初始化过程（填充属性，调用aware子类的方法，调用BeanPostProcessor前置处理方法，调用init-method方法，调用BeanPostProcessor的后置处理方法）
7. 生成完整的bean对象，通过getBean方法可以直接获取
8. 销毁过程

# 谈谈你对Spring IOC底层原理实现的理解

createBeanFactory，getBean，doGetBean，createBean，doCreateBean，createBeanInstance（getDeclaredConstructor，newInstance），populateBean，initializingBean

1. 先通过createBeanFactory创建一个Bean工厂（DefaultListableBeanFactory）
2. 开始循环创建对象，因为容器中的bean默认都是单例的，所有优先通过getBean，doGetBean从容器中查找，找不到的话创建
3. 通过createBean，doCreateBean方法，以反射的方法创建对象，一般情况下使用的是无参构造方法（getDeclaredConstructor，newInstance）
4. 进行对象的属性填充populateBean
5. 进行其他的初始化操作（initializingBean）

# Spring中应用到的设计模式

IOC：

- 单例模式（bean）
- 原型模式（bean）
- 工厂模式
  - 简单工厂：BeanFactory通过参数获取对象
  - 工厂方法：FactoryBean，用户控制生成对象的过程，将工厂交给Spring管理

Context初始化：

- 组合模式：ApplicationContext实现了BeanFactory，内部通过DefaultListableBeanFactory实现真正的功能

- 模板模式：refresh给子类实现，jdbctemplate

- 观察者模式：Event、Listener、Publisher

- 装饰者模式：Wrapper、Decorator 

  动态增加新功能，代替继承的技术

- 策略模式：XmlBeanDefinitionReader/PropertiesBeanDefinitionReader

- 适配器模式：AfterReturningAdviceAdapter、MethodBeforeAdviceAdapter

- 委托者模式

AOP：

- 代理模式
- 责任链模式：AOP中的拦截器链，

# @Autowired

构造方法注入在实例化的时候注入 

```java
// 通过参数的beanName获取参数
argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
      getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
```

字段和方法用@Autowired修饰通过BeanPostProcessor（AutowiredAnnotationBeanPostProcessor初始化容器时加载）

```java
// populateBean方法前面实例化之后会预解析
applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);

protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (MergedBeanDefinitionPostProcessor processor : getBeanPostProcessorCache().mergedDefinition) {
        processor.postProcessMergedBeanDefinition(mbd, beanType, beanName);
    }
}
// populateBean方法中后会查找@Autowired修饰的
for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
				PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
				if (pvsToUse == null) {
					if (filteredPds == null) {
						filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
					}
					pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						return;
					}
				}
				pvs = pvsToUse;
			}
```

- 构造注入的优势
  1. 强制依赖变量初始化：通过构造函数注入，类在实例化时必须注入所有依赖，确保了依赖变量在类实例化时就能够被正确初始化
  2. 不变性：构造函数注入提倡依赖变量的不变性（final），这样可以确保引用一旦被注入，便不会被更改，从而提高了代码的安全性和可维护性
  3. 简化测试：通过构造函数注入，可以方便的进行单元测试而无需启动整个Spring容器，只需传递模拟对象（Mock objects）即可

# 什么是循环依赖

A依赖于B，B依赖于A，创建对象时会无限创建下去

**分析问题**

首先我们要明确一点就是如果这个对象A还没创建成功，在创建的过程中要依赖另一个对象B，而另一个对象B也是在创建中要依赖对象A，这种肯定是无解的，这时我们就要转换思路，先把A创建出来，但是还没有完成初始化操作，也就是这是一个半成品的对象，然后在赋值的时候先把A暴露出来，然后创建B，让B创建完成后找到暴露的A完成整体的实例化，这时再把B交给A完成A的后续操作

构造方法中的循环依赖无法解决

初始化方法的循环依赖可以通过提前暴露解决

# Spring中的循环依赖问题

![](.\pic\spring\spring中循环依赖.jpg)

# Spring中是如何检测构造注入的循环依赖问题的

```java
public boolean isSingletonCurrentlyInCreation(String beanName) {
    return this.singletonsCurrentlyInCreation.contains(beanName);
}
// AbstractBeanFactory中getSingleton方法中调用 单例创建前会调用判断当前beanName是否已经正在创建
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}

// 原型模式亦然
if (isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}

protected boolean isPrototypeCurrentlyInCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    return (curVal != null &&
            (curVal.equals(beanName) || (curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
}
```

# Spring的Bean的生命周期

![](.\pic\spring\bean完整生命周期.jpg)

https://www.bilibili.com/video/BV1584y1r7n6/?spm_id_from=333.337.search-card.all.click&vd_source=584c7c5c42e674dab0341bef94cd689a

![](.\pic\spring\bean的完整周期2.jpg)

- 初始化容器ApplicationContext，初始化内部BeanFactory，loadBeanDefinitions通过Xml或PropertiesBeanDefinitionReader或者注解读取，存放到beanDefinitionMap中
- 注解方式加载bean是靠BeanFactoryPostProcessor的子类BeanDefinitionRegisterPostProcessor的子类ConfigurationClassPostProcessor中的parser.parse方法做的（扩展性强，注解后出的，不是直接改的解析文件）
- 根据beanDefinitionNames开始遍历是单例，非抽象类非懒加载，如果是FactoryBean就走xxx，不是走正常的doGetBean，走createBean
- createBeanInstance实例化对象，通过BeanDefinition中的Class反射获取构造方法
- 填充属性（依赖其他对象，可能又要创建或者从缓存中获取）populateBean()，循环依赖的问题（三级缓存）
- 调用aware接口相关的方法：invokeAwareMethod，先看是否实现三种Aware接口（BeanName，BeanFactory，BeanClassLoader），注入相应的属性
- 执行Bean的初始化前置处理器：使用比较多的有ApplicationContextAwareProcessor，设置ApplicationContext，Environment，ResourceLoader，EmbeddValueResolver等6个对象（因为是set方法，所以可以继续用代码扩展功能，Spring MVC中ApplicationObjectSupport在setApplicationContext之后除了赋值还有initApplicationContext）
- 执行@PostConstruct方法
- 调用initmethod方法，invokeInitMethod()，检查是否实现InitializingBean接口，执行此接口重写的afterPropertiesSet方法，再看看有没有自定义的initMethod方法
- 执行Bean的初始化后置处理器，spring的aop就是在此处实现的，AbstractAutoProxyCreator
- 实现DisposableBean接口或指定destroy-method的bean注册到disposableBeans
- 获取到完整的对象，可以通过getBean的方式进行对象的获取
- 销毁流程：1.判断是否实现了DispoableBean接口 2.调用destroyMethod方法

# Spring的三级缓存在生命周期的位置

![](.\pic\spring\bean的生命周期.jpg)

# 三级缓存的作用

<img src=".\pic\spring\三级缓存.jpg" style="zoom:200%;" />

A -> B -> A

创建A流程：doGetBean()，getSingleton(beanName)中查看一级缓存是否有，有则返回，没有看二级缓存是否有，有则返回，没有看三级缓存是否有，有则执行三级缓存中存的ObjectFactory的getObject方法，如果是代理对象则执行处理器生成代理对象，否则返回原始对象，将对象放入二级缓存，从三级缓存移除（第一次创建没有A），进入if(mbd.isSingleton())中，在getSingleton中执行beforeSingletonCreation(beanName)放入singletonsCurrentlyInCreation并判断是否已经正在创建，之后正常实例化createBeanInstance(beanName, mbd, args)

```java
// 再判断是否单例，是否支持循环依赖，是否正在创建中
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
      isSingletonCurrentlyInCreation(beanName));
// 是的话将ObjectFactory放入三级缓存中
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                "' to allow for resolving potential circular references");
    }
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
// 返回原始对象或者代理对象
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
        }
    }
    return exposedObject;
}
```

populateBean(beanName, mbd, instanceWrapper)发现需要注入B

创建B的流程：同上，populateBean(beanName, mbd, instanceWrapper)发现需要注入A

此时会在三级缓存找到A的原始对象，并把A的原始对象放入二级缓存中，从三级缓存中删除，并返回，B创建完毕，执行afterSingletonCreation(beanName)把B从singletonsCurrentlyInCreation移除，把B从三级缓存和二级缓存中删除，放入一级缓存

回到A的创建流程，同上

## 为什么需要三级缓存

三级缓存的value类型是ObjectFactory，是一个函数式接口，存在的意义是保证在整个容器的运行过程中同名的bean只能有一个

如果一个对象需要被代理，或者说需要生成代理对象，那么要不要优先生成一个普通对象？要

普通对象和代理对象是不能同时出现在容器中的，因此当一个对象需要被代理的时候，就要使用代理对象覆盖掉之前的普通对象，在实际的调用过程中，是没有办法确定什么时候对象被使用，所以就要求当某个对象被调用的时候，优先判断此对象是否需要被代理，类似于一种回调机制的实现，因此传入lambda表达式的时候，可以通过lambda表达式来执行对象的覆盖过程，getEarlyBeanReference()

因此，所有的bean对象在创建的时候都要优先放到三级缓存中，在后续的使用过程中，如果需要被代理则返回代理对象，如果不需要被代理，则直接返回普通对象

## 缓存的放置时间和删除时间

三级缓存：createBeanInstance之后，addSingletonFactory

二级缓存：第一个从三级缓存确定对象是代理对象还是普通对象的时候，同时删除三级缓存getSingleton

一级缓存：生成完整对象之后放到一级缓存，删除二三级缓存：addSingleton

# 谈谈你对Spring中AOP的理解

动态代理

aop是ioc的一个扩展功能，先有的ioc，再有的aop，只是在ioc的整个流程中新增的一个扩展点而已：BeanPostProcessor

总：

​	aop概念，应用场景，动态代理

分：

​	bean的创建过程中有一个步骤可以对bean进行扩展实现，aop本身就是一个扩展功能，所以在BeanPostProcessor的后置处理方法中来进行实现

1. 代理对象的创建过程（advice，切面，切点）通过AbstractAutoProxyCreator的后置增强处理器执行wrapIfNecessary方法判断当前对象是否需要代理（是否符合切面条件等等）
2. 通过jdk或者cglib的方式来生成代理对象，通过proxyFactory.getProxy(classLoader)，spring默认使用cglib生成代理对象（enhancer）
3. 在执行方法调用的时候，会调用到生成的字节码文件中，直接会找到DynamicAdvisedInterceptor类中的intercept方法，从此方法开始执行
4. 根据之前定义好的通知来生成拦截器链，获取所有与该方法匹配的所有“增强方法”，并将它们组成调用链，同时进行排序
5. 从拦截器链中依次获取每一个通知开始进行执行，再执行过程中，为了方便找到下一个通知是哪个，会有一个CglibMethodInvocation的对象，找的时候是从-1的位置依次开始查找并且执行的，执行的过程中会插入真实的bean的方法

# Spring中事务的隔离级别

spring框架的事务管理是基于java的，而数据库的事务隔离级别是由数据库系统本身实现的。Spring事务隔离级别之所以能够与数据库隔离级别不一致，是因为Spring事务管理提供了一个抽象层，它可以在应用程序代码与底层数据库之间创建一个中间层。这种设计允许开发者在应用程序中定义事务行为，而无需直接依赖于特定的数据库系统

Spring定义了以下几种事务隔离级额别：

- DEFAULT

  使用底层数据库的默认隔离级别

- READ_UNCOMMITTED

- READ_COMMITTED

- REPEATABLE_READ

- SERIALIZABLE

一般情况下会设置为READ_COMMITED，此时避免了脏读，并发性也不错，之后哦再通过一些别的手段去解决不可重复读和幻读的问题就好了

# Spring中事务的传播行为

事物的传播行为针对的是嵌套的关系

Spring中的7个事务传播行为：

| 事务行为                  | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 支持当前事务，假设当前没有事务。就新建一个事务               |
| PROPAGATION_SUPPORTS      | 支持当前事务，假设当前没有事务，就以非事务方式运行           |
| PROPAGATION_MANDATORY     | 支持当前事务，假设当前没有事务，就抛出异常                   |
| PROPAGATION_REQUIRED_NEW  | 新建事务，假设当前存在事务，把当前事务挂起                   |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式运行操作，假设当前存在事务，就把当前事务挂起     |
| PROPAGATION_NEVER         | 以非事务方式运行，假设当前存在事务，则抛出异常               |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作 |

某一个事务嵌套另一个事务的时候怎么办

A方法调用B方法，AB方法都有事务，并且传播特性不同，那么A如果有异常，B怎么办，B如果有异常，A怎么办

总：

​	事务的传播特性指的是不同方法的嵌套调用过程中，事务应该如何进行处理，是用同一个事务还是不同的事务，当出现异常的时候会回滚还是提交，两个方法之间的相关影响，在日常工作中，使用比较多的是required，required_new，nested

分：

1. 先说事务的不同分类，可以分为三类：支持当前事务，不支持当前事务，嵌套事务

2. 如果外层方法是required，内层方法是required，required_new，nested

   如果外层方法是required_new，内层方法是required，required_new，nested

   如果外层方法是nested，内层方法是required，required_new，nested

# Spring事务实现的方式

**编程式事务管理：**这意味着你可以通过编程的方式管理事务，这种方式带来了很大的灵活性，但很难维护

**声明式事务管理：**这种方式意味着你可以将事务管理和业务代码分离，只需要通过注解或者XML配置管理事务

# Spring中事务的本质

# Spring的事务是如何回滚的

声明式事务，编程式事务

总：

​	spring的事务是由aop来实现的，首先要生成具体的代理对象，然后按照aop的整套流程来执行具体的操作逻辑，正常情况下要通过通知来完成核心功能，但是事务不是通过通知来实现的，而是通过一个TransactionInterceptor来实现的，然后调用invoke来实现具体的逻辑

分：

1. 先做准备工作，解析各个方法上事务相关的属性，根据具体的属性来判断是否开始新事务
2. 当需要开启的时候，获取数据库连接，关闭自动提交功能，开启事务
3. 执行具体的sql逻辑操作
4. 在操作过程中，如果执行失败了，那么会通过completeTransactionAfterThrowing看来完成事务的回滚操作，回滚的具体逻辑是通过doRollBack方法来实现的，实现的时候也是要先获取连接对象，通过连接对象来回滚
5. 如果执行过程中，没有任何意外情况的发生，那么通过commitTransactionAfterReturning来完成事务的提交操作，提交的具体逻辑是通过doCommit方法来实现的，实现的时候也是获取连接，通过连接对象来提交
6. 当事务执行完毕之后需要清除相关的事务信息cleanupTransactionInfo

如果想聊的更加细致的话，需要知道TransactionInfo，TransactionStatus

https://www.bilibili.com/video/BV1fu411V77w/?spm_id_from=333.337.search-card.all.click&vd_source=584c7c5c42e674dab0341bef94cd689a

# Spring中支持几种作用域

1. prototype

   使用该属性定义Bean时，IOC容器可以创建多个Bean实例，每次返回都是一个新的实例

2. singleton

   使用该属性定义Bean时，IOC容器仅创建一个Bean实例，IOC容器每次返回的是同一个Bean实例

3. request

   该属性进对HTTP请求产生作用，使用该属性定义Bean时，每次HTTP请求都会创建一个新的Bean，适用于WebApplicationContext环境

4. session

   该属性仅用于HTTP Session，同一个Session共享一个Bean实例。不同Session使用不同的实例

5. global-session

   该属性仅用于HTTP Session，同session作用域不同的是，所有的Session共享一个Bean实例

# ApplicationContext和BeanFactory的区别

ApplicationContext

- BeanFactory

  - DefalutListableBeanFactory -> 各种Post处理
  - 组合模式

- Environment

  Apollo -》 初始化配置，好像可以不用在application.properties里配置，可以配置在Apollo

- MessageSource

  国际化

- EventPublish

  领域事件

  - 领域，主链路
  - 注意重启

- Resource

  resources目录 -> 工具

# 谈谈对BeanFactoryPostProcessor的理解

对BeanFactory的后置处理

对BeanDefinition进行修改

解析@Configuration注解的对象

# 谈谈对BeanPostProcessor的理解

一个用于对bean进行增强的接口，实现他的类注入到IOC容器之后就可以在初始化之前对bean增强和初始化之后对bean增强

```java
@Component
public class LicenseBeanProcessor implements BeanPostProcessor {
    private final Logger logger = LoggerFactory.getLogger(LicenseBeanProcessor.class);
    private static final String DUBBO_CLASS_NAME = "com.alibaba.dubbo.common";
    public static final Map<String, LicenseMethodConfig> DUBBO_PROTOCOL_RELATIONS = new HashMap();
    public static final Map<String, LicenseMethodConfig> T3_PROTOCOL_RELATIONS = new HashMap();

    public LicenseBeanProcessor() {
    }

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        Class<?> clazz = bean.getClass();
        if (clazz.getName().startsWith("com.alibaba.dubbo.common")) {
            return bean;
        } else {
            if (this.isProxyBean(bean)) {
                clazz = AopUtils.getTargetClass(bean);
            }

            Class<?>[] interfaces = clazz.getInterfaces();
            if (interfaces != null && interfaces.length != 0) {
                Class[] var5 = interfaces;
                int var6 = interfaces.length;

                for(int var7 = 0; var7 < var6; ++var7) {
                    Class<?> c = var5[var7];
                    CloudService cloudService = (CloudService)AnnotatedElementUtils.findMergedAnnotation(c, CloudService.class);
                    if (cloudService != null && !cloudService.singleMode()) {
                        Method[] methods = c.getMethods();
                        Method[] var11 = methods;
                        int var12 = methods.length;

                        for(int var13 = 0; var13 < var12; ++var13) {
                            Method method = var11[var13];
                            String dubboFunctionId = c.getName() + "." + method.getName();
                            String t3FunctionId = null;
                            CloudFunction cloudFunction = (CloudFunction)method.getAnnotation(CloudFunction.class);
                            if (cloudFunction != null) {
                                t3FunctionId = StringUtils.isNotEmpty(cloudFunction.value()) ? cloudFunction.value() : cloudFunction.functionId();
                            }

                            t3FunctionId = StringUtils.isNotEmpty(t3FunctionId) ? t3FunctionId : dubboFunctionId;
                            Class checklessClass = null;
                            Class joinLicenseBlackListClass = null;

                            try {
                                checklessClass = Class.forName("com.hundsun.jrescloud.rpc.annotation.license.Checkless");
                                joinLicenseBlackListClass = Class.forName("com.hundsun.jrescloud.rpc.annotation.license.JoinLicenseBlackList");
                            } catch (ClassNotFoundException var23) {
                                if (this.logger.isDebugEnabled()) {
                                    this.logger.debug("LicenseBeanProcessor [Checkless、JoinLicenseBlackList] ClassNotFoundException");
                                }
                            }

                            Class joinLicenseApiGroupClass = null;

                            try {
                                joinLicenseApiGroupClass = Class.forName("com.hundsun.jrescloud.rpc.annotation.license.LicenseApiGroup");
                            } catch (ClassNotFoundException var22) {
                                if (this.logger.isDebugEnabled()) {
                                    this.logger.debug("LicenseBeanProcessor [com.hundsun.jrescloud.rpc.annotation.license.LicenseApiGroup] ClassNotFoundException");
                                }
                            }

                            this.licenseMethodConfigWrapper(checklessClass, joinLicenseBlackListClass, joinLicenseApiGroupClass, dubboFunctionId, t3FunctionId, method);
                        }
                    }
                }

                return bean;
            } else {
                return bean;
            }
        }
    }

    private boolean isProxyBean(Object bean) {
        return AopUtils.isAopProxy(bean);
    }

    private void licenseMethodConfigWrapper(Class checklessClass, Class joinLicenseBlackListClass, Class joinLicenseApiGroupClass, String dubboFunctionId, String t3FunctionId, Method method) {
        if (joinLicenseApiGroupClass != null && checklessClass != null && joinLicenseBlackListClass != null) {
            // 丢到了T3_PROTOCOL_RELATIONS中
            LicenseMethodConfigWrapper.getInstance().wrapper2(method, t3FunctionId, dubboFunctionId);
        } else if (checklessClass != null && joinLicenseBlackListClass != null) {
            LicenseMethodConfigWrapper.getInstance().wrapper(method, t3FunctionId, dubboFunctionId);
        } else {
            LicenseMethodConfigWrapper.getInstance().wrapper(t3FunctionId, dubboFunctionId);
        }

    }
}
```

# Spring中的单例是线程安全的吗

不是线程安全的，单例对象只要是无状态（不保存数据）的就是线程安全的

如需保证线程安全，可以加锁、CAS、lock，或者存储在ThreadLocal中

> Spring的事务管理器使用ThreadLocal为不同线程维护了一套独立的connection副本，保证线程之间不会互相影响（Spring是如何保证事务获取同一个Connection的）

# Spring为啥默认单例

提高性能，少创建实例，少占用空间，少垃圾回收，缓存快速获取

# BeanFactory和FactoryBean有什么区别

- BeanFactory

  负责生产和管理Bean的一个工厂接口，是Spring中IOC的核心接口，提供了一个Spring IOC的容器规范

- FactoryBean

  也是一个接口，是一个bean，是一个能生产或者修饰对象生成的工厂bean，spring提供的另外一种bean的创建方式，可以让我们自定义bean的创建过程，对bean的一种拓展

相同点：都是用来创建bean对象的

不同点：使用BeanFactory创建对象的时候，必须要遵循严格的生命周期流程，太复杂了，如果想要简单的自定义某个对象的创建，同时创建完成的对象想交给spring来管理，那么就需要实现FactoryBean接口了

# @Autowired和@Resource的区别

都是用来Spring中自动装配的注解，可以放在字段上或者set方法上

## @Autowired

是Spring提供的注解，默认情况下是以byType的方式注入，需要对象必须存在，如果允许为null值，设置required为false，如果想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用

## @Resource

是J2EE的注解，默认情况下是按照byName的方式注入，@Resource有两个重要的属性：name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略

自动装配Mapper时@Autowired会报错，@Resource不会报错，可能是因为@Mapper是mybatis的注解，Spring识别不出是否一定有值

# Spring中常用注解

@Controller、@Service、@Compenent、@Autowired、@Bean、@Import

# 谈谈对@Indexed注解的作用

@Indexed注解是Spring5.0提供

@Indexed注解解决的问题：是随着项目越来越复杂，那么@ComponentScan需要扫描加载的Class会越来越多，在系统启动的时候会造成性能损耗，所以@Indexed注解的作用其实就是提升系统启动的性能

要先引入pom spring-context-indexer

在系统编译的时候会收集所有被@Indexed注解标识的java类。然后记录在META-INF/spring.components文件中。那么系统启动的时候就只需要读取一个该文件中的内容就不用再遍历所有的目录了。提升了效率

Spring5.0之后@Componet注解默认带@Indexed注解

# @Component、@Controller、@Service、@Repository的区别

@Componet：将java类标记为bean，它是任何Spring管理组件的通用结构型。spring的组件扫描机制可以将其拾取并拉入容器中

@Controller：将一个类标记为Spring Web MVC控制器，标有他的bean会自动导入到IOC容器，底层就是@Component

@Service：此注解是组件注解Component的特化。他不会对@Component注解提供任何其他行为。可以在服务层类中使用@Service而不是@Component，因为它以更好地方式指定了意图

@Repository：这个注解是具有类似用途和功能的@Component注解的特化。他为DAO提供了额外的好处，他将DAO导入IOC容器，并使未经检查的异常有资格转为Spring DataAccessException
