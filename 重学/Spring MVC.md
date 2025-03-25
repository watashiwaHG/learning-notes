# Spring MVC在Spring中的扩展点

## postProcessorBeanFactory

这是Spring中AbstractApplicationContext的模板方法，被MVC的子类AbstractRefreshableWebApplicationContext实现了，添加了个BeanPostProcessor

# 谈谈对Spring MVC的理解

Spring MVC是属于Spring framework生态里面的一个模块，他是在servlet上面构建并且使用了MVC模式，设计的一个web框架。他的目的是为了简化传统的servlet加jsp模式下的web开发方式，其次Spring MVC的整个架构设计是对java web里面的MVC的框架模式做了一些增强和扩展，在开发mvc应用的时候会更加的方便和灵活

# Spring MVC

**启动过程**：

1. 启动tomcat

2. 解析web.xml（<listener> <servlet>）

3. ContextLoaderListener.init() 创建Spring容器 ---> ServletContext（父容器）

4. 创建并初始化DispatcherServlet

5. - DispatcherServlet的（HttpServletBean）init() 

   - 创建spring容器（将父容器设置，添加了一个监听器ContextRefreshListener） 

   -  在refresh之后监听器执行initStrategies(context) 

     实质上就是DispatcherServlet的初始化方法（初始化Spring MVC中的9大组件）

   - initHandlerMappings(context)

     getDefaultStrategies(context, HandlerMapping.class)从DispatcherServlet.properties文件中读取到默认的HandlerMapping

   - 创建bean放入容器中（触发afterPropertiesSet方法）

     RequestMappingHandlerMapping中initHandlerMethods()找到容器中实现了@Controller或者@RequestMapping注解的bean，找到bean中含有@RequestMapping注解的方法，将方法和他的类做一个map映射，最后注册到mappingRegistry（url -> 方法）
     
   - RequestMappingHandlerAdapter.afterPropertiesSet() 初始化@ControllerAdvice（@Component） @InitBinder（类型转换器，字符串转Date）@ModelAttribute

**请求处理过程：**

DispatcherServlet ---> 接收请求 ---> path，request parameters ---> path ---> @RequestMapping Method ---> 解析参数，参数绑定 ---> 执行方法 ---> 返回值解析

1. 先从三种handlerMapping中根据路径找到所映射的handler（类或者方法信息，放在责任链中，拦截器）

   ```java
   mappedHandler = getHandler(processedRequest);
   ```

2. 根据handler找到匹配的handlerAdapter，判断到底是实现了Controller接口（SimpleControllerHandlerAdapter），还是实现了HttpRequestHandler接口（HttpRequestHandlerAdapter），还是HandlerMethod（RequestMappingHandlerAdapter）

   ```java
   // Determine handler adapter for the current request.
   HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
   ```

3. 执行拦截器的前置方法

   ```java
   if (!mappedHandler.applyPreHandle(processedRequest, response)) {
      return;
   }
   ```

4. 执行真正的方法handlerAdapter，并且返回ModelAndView对象（可能是视图，也可能是字符串）

   ```java
   // Actually invoke the handler.
   mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
   ```

   RequestMappingHandlerAdapter

   执行方法前，对不同的参数获取相应的参数解析器argumentResolvers解析

   执行方法后，对返回值进行解析

5. 执行拦截器的后置方法

   ```java
   mappedHandler.applyPostHandle(processedRequest, response, mv);
   ```

6. ViewResolver视图解析器解析，并返回一个View对象，接着DispatcherServlet会将视图渲染到response对象中

7. 执行拦截器的afterCompetition方法

   ```java
   mappedHandler.triggerAfterCompletion(request, response, ex);
   ```

![](./pic/spring%20mvc/spring%20mvc执行流程.png)

# handler是什么

能够处理请求的bean对象或者方法

- 添加了@RequestMapping注解的方法
- 实现了Controller接口的bean对象
- 实现了HttpRequestHandler接口的bean对象
- RouterFunction类型的bean对象

# HandlerMapping是什么

HandlerMapping是Spring MVC中的一个非常重要的组件，主要用来保存请求路径和处理器之间的映射关系，本质上是一个map，key为请求路径（path），value为其对应的请求处理器（Handler）

在Spring MVC启动过程中，会完成对HandlerMapping的初始化，默认会加载如下3个HandlerMapping并将其创建为bean对象：

- BeanNameUrlHandlerMapping

  负责寻找映射实现了Controller接口或HttpRequestHandler接口的Handler对象

- RequestMappingHandlerMapping

  负责寻找映射使用了@RequestMapping注解的方法的Handler对象

- RouterFunctionMapping

  负责寻找并映射所有的RouterFunction类型的Handler对象

# HandlerAdapter是什么

HandlerAdapter同样是Spring MVC中非常重要的一个组件，它主要用来执行相应的handler，从名称上来看，HandlerAdapter采用了适配器模式，以适配不同类型的Handler的执行，Spring MVC中，HandlerAdapter接口的定义如下：

support：判断当前适配器是否适配待处理的handler

handle：执行handler

在Spring MVC启动过程中，会完成HandlerAdapter的初始化，如果我们没有事先定义我们自己得HandlerAdapter，Spring MVC会默认加载如下4个HandlerAdapter并将其创建为bean对象：

- HttpRequestHandlerAdapter

  是为了适配HttpRequestHandler接口类型的Handler

- SimpleControllerHandlerAdapter

  是为了适配Controller接口类型的Handler

- RequestMappingHandlerAdapter

  是为了适配使用了@RequestMapping注解的方法的Handler

- HandlerFunctionAdapter

  是为了适配RouterFunction类型的Handler

在Spring MVC处理请求的过程中，会根据HandlerMapping获取到的Handler匹配相应的HandlerAdapter，该HandlerAdapter会通过参数解析器解析所有的参数，然后通过反射执行Handler，执行结果经过返回值处理器处理后会返回一个ModelAndView对象
