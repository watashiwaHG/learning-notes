# SpringBoot的启动流程

- 实例化一个SpringApplication对象
  1. run方法传入进来的类赋值给primarySource（数组，可以有多个）
  2. 推测应用的类型（通过是否能加载相应的类判断是Servlet，Reactive，None）
  3. 从spring.factories文件中获取ApplicationContextInitializer的实现类，赋值给initializers
  4. 从spring.factories文件中获取ApplicationListener的实现类，赋值给listeners
  5. 找到main方法所在的类
- 调用SpringApplication对象的run方法
  1. 从spring.factories文件中获取SpringApplicationRunListener的实现类（这个是SpringBoot自己的监听器，主要监听SpringBoot的run方法中的各个节点）
  2. 调用所有SpringApplicationRunListener实现类的starting方法（默认监听器会发布一个starting事件给ApplicationListener）
  3. 准备Environment，将配置文件中的key value封装起来
  4. 调用所有SpringApplicationRunListener实现类的environmentPrepared方法（默认监听器会发布一个starting事件给ApplicationListener，可以在这里对environment进行修改）
  5. 打印banner
  6. 根据前面推测的应用类型创建Spring相应的容器
  7. 执行ApplicationContextInitializer的initialize方法初始化容器
  8. 调用所有SpringApplicationRunListener实现类的contextPrepared方法（默认监听器会发布一个contextPrepared事件给ApplicationListener）
  9. load方法向Spring容器中添加配置类（run方法传入进来的类，也就是primarySource，通过AnnotatedBeanDefinitionReader读取）
  10. 调用所有SpringApplicationRunListener实现类的contextLoaded方法（默认监听器会发布一个contextLoaded事件给ApplicationListener）
  11. 启动Spring容器，调用refresh方法
  12. 调用afterRefresh方法 -->模版方法的设计模式
  13. 调用所有SpringApplicationRunListener实现类的started方法（默认监听器会发布一个started事件给ApplicationListener）
  14. callRunners方法，调用所有实现了ApplicationRunner和CommandLineRunner的run方法，传入的值为命令行中的参数，ApplicationRunner传入的参数功能更加丰富
  15. 如果这个过程报错调用所有SpringApplicationRunListener实现类的failed方法（默认监听器会发布一个failed事件给ApplicationListener）
  16. 调用所有SpringApplicationRunListener实现类的ready方法（默认监听器会发布一个ready事件给ApplicationListener）

# SpringBoot的自动装配

SpringBoot在启动的过程中，

@SpringBootApplication --》 @EnableAutoConfiguration --》@Import(AutoConfigurationImportSelector.class) --》selectImports() --》spring.factories --》spring-autoconfigure-metadata.properties过滤

# SpringBoot在Spring中的扩展点

**onRefresh**

在SpringBoot中ServletWebServerApplicationContext重写了AbstractApplicationContext的onRefresh方法，createWebServer创建了Tomcat容器

# 如果要对属性文件中账号密码进行解密如何实现

![](E:\转正\学习\study note\重学\pic\spring\密码解密.jpg)

实现EnvironmentPostProcessor接口的postProcessEnvironment方法可能拿到配置文件的信息，对密码字段进行解密就可以

# 为什么SpringBoot的jar可以直接运行

1. SpringBoot提供了一个插件spring-boot-maven-plugin用于把程序打成一个可执行的jar包（生成了manifest文件，里面指定了Main-Class以及Start-Class）
2. SpringBoot应用打包之后，生成一个Fat jar（jar包里包含jar），包含了应用依赖的jar包和Spring Boot loader相关的类
3. java -jar会去找jar中的manifest文件（这是JVM的规范），在那里找到真正的启动类（Main-Class）
4. Fat jar的启动函数是JarLauncher，他负责创建一个LaunchURLClassLoader来加载boot-lib下面的jar，并以一个新线程启动应用的启动类的Main函数（找到manifest中的Start-Class，通过反射执行Start-Class中的Main方法）

# SpringBoot为什么@Condition中没有引入的jar包不会报错

首先在SpringBoot.jar中他是引入了这些jar包的，是通过maven的<optional>标签来让使用方选择性引入，所以比如默认情况下使用方会引入tomcat的jar包，如果想使用jetty则需要先排除掉tomcat的jar包，然后手动引入jetty的jar包才能使用，而SpringBoot的jar包已经是编译过的class文件，所以不会再次编译检查这些包是否已经引入，也就不会报错了

# SpringBootApplication注解的作用

- @SpringBootConfiguration

  里层就是@Configuration注解，在启动类的SpringApplication的run方法中传入当前启动类，会把启动类当作配置类（xml文件）加载到Spring容器中

- @ComponentScan

  默认扫描启动类所在目录下的所有文件（将用户程序中定义的bean加载到Spring容器中）

- @EnableAutoConfiguration

  里层包含@Import注解，这是一个用来加载bean到Spring容器的注解，加载的AutoConfigurationImportSelector.class类实现了ImportSelector接口，重写了selectImports方法，读取项目中META-INF中的spring.factories文件，读取其中EnableAutoConfiguration的值（在springboot2.7版本之后，将内容转移到了另外一个单独的文件中了META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports，不需要读取整个文件再从中筛选EnableAutoConfiguration，但也保留了从META-INF中的spring.factories读取的方法，可以读取到其他非SpringBoot jar包中的spring.factories文件），经过filter过滤掉不需要加载的bean（首先通过asm的方式读取到所有类上的各种@Condition注解，而不需要先将类加载进来再去读取@Condition注解，通过@Condition使用类加载或者从容器中获取bean或者从Environment中获取配置文件的值等等方式判断），最后将需要加载的bean加载到容器中（将SpringBoot中帮我们定义好的bean自动装配到容器中）

# SpringBoot中的Starter的作用

Starter主要分为SpringBoot自己的starter和第三方的starter，核心思想都是其中的pom文件，帮助我们引入使用这个功能需要引入的所有依赖，Starter中是不包含代码的，而第三方的Starter还会引入自己的AutoConfiguration包，将需要自动装配的bean配置到META-INF中的spring.factories，通过SpringBoot的自动装配加载到容器中

# SpringBoot的配置文件读取顺序

默认是application开头的文件，可以修改，properties文件优先于yml文件，config目录下的文件优先于外面的文件，config下一层目录下的配置文件优先于外面的配置文件，file目录下的文件优先于classpath下的文件

# SpringBoot整合Tomcat

spring-boot-starter-web默认引入了spring-boot-starter-tomcat，使用Tomcat。ServletWebServerApplicationContext重写了AbstractApplicationContext的onRefresh方法，createWebServer创建Web容器，getWebServerFactory获取ServletWebServerFactory，其实就是从Spring容器中获取ServletWebServerFactory，ServletWebServerFactory是ServletWebServerFactoryAutoConfiguration中自动加载的，默认把三种WebServerFactory全部导入，但会根据是否能加载到相应的类来判断是否真的加载到Spring容器中

# SpringBoot整合Mybatis

首先引入mybatis的starter包

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>
```

这个starter中主要引入的有

```xml
<dependency>
	<groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
</dependency>
<dependency>
	<groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
</dependency>
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
</dependency>
```

其中autoconfigure是整合到SpringBoot中主要的包，在META-INF中的spring.factories下引入MybatisAutoConfiguration类，帮助用户生成SqlSessionFactory，需要传入的DataSource由SpringBoot的DataSourceAutoConfiguration生成，DataSource需要传入的DataSourceProperties与配置文件中spring.datasource前缀的配置绑定，MybatisAutoConfiguration引入AutoConfiguredMapperScannerRegistrar扫描启动类所在包下的含有@Mapper的接口，如果不加@Mapper注解，可以使用@MapperScan加包名扫描所有的Mapper（引入了MapperScannerRegister）

# SpringBoot整合AOP

首先引入AOP的starter包

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

这个starter中主要引入的有

```xml
<dependency>    
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
</dependency>
<dependency>    
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

SpringBoot本身有AopAutoConfiguration类，默认情况下开启AOP，引入AOP的包后就可以加载JdkDynamicAutoProxyConfiguration或者CglibAutoProxyConfiguration，这两个类通过spring.aop.proxy-target-class的值来控制加载哪个，默认是CglibAutoProxyConfiguration，这两个类本身没有什么功能，主要是使用的注解@EnableAspectJAutoProxy的proxyTargetClass值不同，CglibAutoProxyConfiguration为true，表示无论什么类都通过cglib创建代理对象，JdkDynamicAutoProxyConfiguration为false，表示如果实现了接口通过Jdk创建代理对象，如果没有实现接口则用cglib创建代理对象

# 配置文件相关注解

**@Value**

直接绑定配置文件的某个属性

**@Environment**

Environment是Spring Core中的一个用于读取配置文件的类，将此类使用@Autowired注入到类中就可以使用他的getProperty方法来获取某个配置项的值

**@PropertiesSource**

指定一个文件作为配置文件

**@ImportResource**

指定一个xml文件作为Spring的配置文件

**@ConfigurationProperties**

直接与配置文件中的多个属性一一绑定，可以指定prefix前缀名

- 与@Component一起使用

  需要作为一个bean在Spring容器中才能生效

- 与@EnableConfigurationProperties一起使用

  通过其他的类去引入@ConfigurationProperties的类，可以在其他的类上添加条件，达到控制是否加载到容器的作用

- 与@ConfigurationPropertiesScan一起使用

  指定包名，扫描包下的所有含有@ConfigurationProperties的类

# 指定不同环境下的配置文件

配置文件或者命令行中设置spring.profile.active的值

```yaml
spring:
	profile:
		active: dev # 代表目前的环境是dev，application-dev.yaml生效，@Profile("dev")生效
```

# 什么是SpringBoot

SpringBoot是由Pivotal团队提供的基于Spring的全新框架，旨在简化Spring应用的初始搭建和开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置（约定大于配置，自动装配）





























