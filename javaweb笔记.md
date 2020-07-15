# Tomcat
## 目录结构
bin: 可执行程序，tomcat在这里启动，startup.bat 启动命令（双击启动），shutdown.bat（双击关闭）

config:Tomcat的配置文件目录

    server.xml:tomcat的配置集中地方，端口配置....
    web.xml:tomcat所有web项目都需要遵循的一个xml

lib：tomcat运行期间需要用到的jar，比如el表达式解析jar

logs：tomcat运行日志

temp：临时文件夹

webapps：集合tomcat中所有运行的项目，每一个项目都是一个文件夹的形式

work：存放运行期间编译的一些东西，比如jsp界面
## 在eclipse中
window-->server-->runtime tomcat的整合信息
## Http（熟悉）
1.f12会看请求头和请求体

2.知道发送请求的整个数据叫（请求报文）

服务器传给浏览器的数据（响应报文）

    报文头部
    空行
    报文主体

请求报文：

    请求首行
    请求头
    空行
    请求体
响应报文：

    响应首行
    响应头
    空行
    响应体
去网上查询http各种响应头，请求头的作用

# Servlet
用来接受请求，处理请求，完成响应的一段小程序
## Servlet体系
Servlet是javaweb的三大组件之一（Servlet、Filter、Listener）
Servlet：接口

HttpServlet：实现类

    doGet()
    doPost()
## Servlet的使用
1.编写Servlet

2.在web.xml中进行配置
    
    <servlet>
        <servlet-name>MyFirstServlet</servlet-name>
        <servlet-class>com.hg.servlet.MyFirstServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>MyFirstServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
## 重要的几个知识点
### ServletConfig
一个Servlet对应一个ServletConfig封装当前Servlet的配置信息
### ServletContext
四大域对象之一（PageContext、ServletRequest、HttpSession、ServletContext）

ServletContext：

作用：

1. 域对象
2. ServletContext获取当前项目信息 ，一个项目对应一个ServletContext，只是代表当前项目
```java
//只能获取当前项目下的某个资源的路径
context.getRealPath("/pics/aaa.jpg");
```
查j2eeAPI文档
### HttpServletRequest
HttpServletRequest：代表请求

每一次发送请求，请求的详细信息会被tomcat自动封装成一个request对象，我们以后要获取当前请求的一些信息，我们使用request对象即可

作用：

1) 获取请求参数：request.getParameter("hello");
2) 作为域对象保存数据，同一次请求期间可以共享数据
3) 获取到HttpSession对象：request.getSession();
4) 转发：request.getRequestDispatcher("/index.jsp").forward(request,response);将当次的请求和响应交给另外一个程序处理，在服务器内部进行的

页面上的img，link(css)都是向服务器发请求

    //给服务器发请求
    <script src="jquery.js"> </script>

### HttpServletResponse
HttpServletResponse：响应

作用：

&nbsp;&nbsp;&nbsp;&nbsp;交给浏览器的数据（响应首行、响应头（对浏览器的一些命令）和响应体（浏览器收到要解析的数据））

&nbsp;&nbsp;&nbsp;&nbsp;重定向：浏览器收到重定向命令以后要发送新请求

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在响应头中会有一个名为Location的字段，告诉浏览器发送新请求的地址
```java
    response.sendRedirect("重定向的地址");
    response.sendRedirect(request.getContextPath()+"/index.jsp");
    request.getContextPath();//获取到项目名，以/开始，不以/结束
```
请求index.jsp页面：servlet index.jsp-->index.jsp.class-->response.getWriter().write("");

回归到底层：服务器给浏览器发送数据，都是写出字符流或者字节流
## 重要知识点
### 乱码
1.请求乱码（浏览器发送给服务器的数据，服务器收到解析出来乱码）

GET、POST：页面上除过表单method="post"外剩下都是get请求
```javascript
    <a href="hello?username=张三">hello</a>
```
Tomcat服务器默认的编解码格式就是ISO-8859-1

GET请求乱码：

原因：所有的请求参数是带在url地址上的，tomcat收到这个请求就会调用默认的编解码格式（ISO-8859-1）将其解码完成，并封装成request对象
```java
    String un = request.getParameter("username");
    un = new String(un.getBytes("ISO-8859-1"),"UTF-8");//原理
```
解决：去改服务器的配置文件：server.xml

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在8080端口配置处添加一句 URIEncoding="utf-8"

POST请求乱码：

原因：请求带来的数据都在请求体中放着，tomcat并不着急解析请求体，一旦调用request.getParameter()，将整个请求体使用默认编码格式全部解析完成

解决：在调用request.getParameter()之前request.setCharacterEncoding("utf-8");（告诉tomcat请求体的数据使用utf-8）

2.响应乱码（服务器发给浏览器的数据，浏览器收到，解析乱码）

原因：直接写出去的数据，浏览器并不知道数据的内容类型以及编码格式等。浏览器用默认的格式打开（utf-8，gbk）

解决：给浏览器的数据一定要说清楚这是什么
```java
    //设置相应头
    response.setContentType("text/html;charset=utf-8");//文本  charset字符集
```
标配：

&nbsp;&nbsp;&nbsp;&nbsp;Tomcat安装整合好，给server.xml8080端口配置处中加上“URIEncoding="UTF-8"”，解决所有GET乱码

&nbsp;&nbsp;&nbsp;&nbsp;Filter： /* 解决POST乱码
```java
    CharacterEncodingFilter{
        requeset.setCharacterEncoding("UTF-8");
        chain.doFilter(request,response);
    }
```
### 路径
#### 相对路径(相对于当前资源所在的路径为标准)
不以/开始的路径就是相对路径
#### 绝对路径（file://c://dd/dd.txt,http://localhost:88/aaa.jsp）
以/开始的路径

区分哪些是服务器解析的路径哪些是浏览器解析的路径
```java
response.sendRedirect("/index.jsp");//因为这个路径是要浏览器访问的，加上项目名
```
只要是html标签里面写的路径都是浏览器解析，除此之外都是服务器解析

    <jsp:forward></jsp:forward>(jsp自定义的标签，目前见到的所有有前缀的标签都是服务器解析的)
#### 服务器解析的路径
```java
request.getRequestDispatcher("/index.jsp");//就是当前项目下的jsp，项目名是自动加上的
```
#### 浏览器解析的路径
在前面拼上服务器地址才是最终要去的路径，页面要写以/开始的路径并且加上项目名

以后推荐写绝对路径，加上项目名
```java
ServletContextListener{
    context.setAttribute("ctp",request.getContextPath());
}
```
```html
<a href="<%=request.getContextPath()%>/index.jsp">hello</a>
<a href="${ctp}/index.jsp">hello</a>
```
# Cookie
## 特点
1) 保存少量数据
2) 都是纯文本
3) 保存的当前网站的cookie，每次访问这个网站都会携带
4) 默认不支持中文
## 使用
1.服务器如何给浏览器发送保存cookie
```java
Cookie cookie = new Cookie("username","hg");
//会在Response Headers中多一条Set-Cookie:username=hg的数据，命令浏览器保存一个cookie
response.addCookie(cookie);
//response.setHeader("Set-Cookie","username=hg");
```
浏览器一旦保存cookie以后，访问这个网站都会带上这个cookie

请求头中会有cookie的键值对

2.cookie有效时间

默认：session 会话期间有效（浏览器只要不关，cookie就在，cookie存在于浏览器的进程中）

setMaxAge(int expiry):设置cookie的有效时间，秒为单位

一个正数：表示多少秒后超时（cookie自动销毁）

一个负数：表示cookie就是会话cookie，随浏览器同生共死

0：cookie立即失效

3.修改或者删除cookie

同名cookie覆盖

4.读取cookie

```java
Cookie[] cookies = request.getCookies();
if(cookies != null){
    for(Cookie cookie : cookies){
        String key = cookie.getName();
        if("username".equals(key)){
            System.out.println(cookie.getValue());
        }
    }
}
```
# Session
服务器端保存当前会话大量数据的一种技术

Session保存的数据是可以在同一个会话期间共享
```java
session.setAttribute("username","zhangsan");
session.getAttribute("username");
```
## 会话控制
为什么再别的地方给session中保存的数据，在另外一个地方可以获取出来

去银行存钱，取钱一致
![](./复习/会话流程图.png)
预先：

1）我们和服务器进行交互期间，可能需要保存一些数据，服务器就为每个会话专门创建一个map，这个map用来保存数据，这个map我们就叫session

2）100个会话就有100个map，每次创建map的时候，这个map有一个唯一标识（JSESSIONID，会话id）

利用浏览器每次访问会带上他所有的cookie

服务器只需要创建一块能保存数据的map，给这个map一个唯一标识（JSESSIONID），创建好以后命令浏览器保存这个map的标识

以后浏览器访问就会带上这个map的标识，服务器就按照标识找到这个map，取出这个map中的数据

特别：

1）cookie失效：默认是cookie没了，通过cookie持久化技术继续找到之前的session

2）session失效：自动超时，手动失效
## 令牌机制
f5将之前的请求再次发出去，表单重复提交

令牌机制：

虎符：

访问页面的时候，生成一个令牌


第一次访问页面生成一个令牌，只要不断刷新页面，f5重新发送上一次请求，令牌不会变化。servlet第一次收到令牌进行比对，比对完毕，更换或者删除令牌，下一次直接去刷新发送上次的请求，由于带来的令牌还是上次的，而这个令牌以及失效了

应用场景：防止表单重复提交，验证码
```java
index.jsp{
    <%
        String token = UUID.randomUUID().toString();
        //分给两处
        //1.一处服务器保存，可以拿到
        session.setAttribute("token",token);
        //2.一处页面保存，每次发请求带上
    %>

    <form action="submit">
        <input name="token" value="<%=token%>"/>
    </form>
}

SumbitServlet{
    //1.拿到服务器保存的令牌
    String token1 = session.getAttribute("token");
    //2.拿到页面带来的令牌
    String token2 = request.getParameter("token");
    if(token1.equals(token2)){
        //处理请求
        session.setAttribute("token","a");
    }else{
        //拒绝处理
    }
}
```
# Filter（过滤器）
Servlet、Filter、Listener（三大组件）

Servlet：处理请求

Filter：过滤拦截请求

Listener：监听器

三大组件基本都需要在web.xml中进行注册，除过Listener中的两个（活化钝化监听器，绑定解绑监听器）需要JavaBean实现，不注册外，剩下的三大组件都需要注册
## 使用
过滤器的使用步骤：

1.实现Filter接口

2.去web.xml进行配置
## Filter配置
```xml
<filter>
    <filter-name>MyFirstFilter</filter-name>
    <filter-class>com.hg.MyFirstFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>MyFirstFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
### url-pattern的三种写法
1.精确匹配

/pics/haha.jsp

直接拦截指定的路径

2.路径匹配（模糊匹配）

/pics/*

拦截pics下的所有请求

3.后缀匹配（模糊匹配）

*.jsp

拦截所有以.jsp结尾的请求

**/pics/*.jsp不满足以上任意一种写法**
## Filter原理
```java
doFilter{
    //放行请求
    chain.doFilter(request,response);
}
```
![](./复习/Filter原理.png)
# Listener（监听器）
javaWeb的三大组件之一（Servlet、Filter、Listener）

八个监听器：
ServletRequest（2）、HttpSession（4）、ServletContext（2）

2：生命周期监听器、属性变化监听器
4（HttpSession）：2+额外的两个（活化钝化监听器、绑定解绑监听器）

掌握的监听器：

ServletContextListener:(生命周期监听器)监听ServletContext的创建和销毁（监听服务器的启动、停止），服务器启动为当前项目创建ServletContext对象，服务器停止销毁创建的ServletContext

ServletContext：

1.一个web项目对应一个ServletContext，他代表当前web项目的信息

2.还可以作为最大的域对象在整个项目运行期间共享数据
## 用法
1.实现对应的监听器接口

2.去web.xml中进行配置

注意：有两个Listener是JavaBean需要实现的接口（HttpSessionActivitionListener，HttpSessionBindingListener）
# Ajax异步请求
![](./复习/ajax原理.png)
***
***
***
# Spring