---
layout: post
title:  "servlet和jsp"
date:   2018-06-02 10:10:30 +0200
categories: web
---

servlet/jsp是服务器端编程的基础

## servlet
servlet是sun(oracle)公司制定的一种用来扩展web服务器功能的组件规范。
### 相关概念
* 扩展web服务器功能：web服务器只能够处理静态资源的请求(是先写好的静态页面)，不能够处理动态资源的请求(需要进行计算，然后生成相应的页面)，所以需要扩展web服务器功能.
* web服务器收到请求之后，如果需要计算，则调用servlet容器来处理，servlet容器会调用servlet来计算
    * 组件规范：符合规范，实现部分功能，需要部署到相应的容器里面才能运行的软件模块
    * 容器：符合规范，提供组件的运行环境的程序。servlet是一个组件，需要部署到相应的容器里面才能运行；如（Tomcat）

### 如何写一个servlet
1. 写一个java类，实现Servlet接口或者继承HttpServlet
2. 编译
3. 打包：appname(应用名)
4. 部署：将3中创建好的文件夹拷贝到容器指定的某文件夹下。也可将3中创建好的文件夹使用jar命令压缩成.war为后缀的文件，然后再拷贝
5. 启动容器，访问servlet。http://ip:port/appname/url-pattern

### servlet如何运行
1. 浏览器依据ip,port建立与web服务器之间的链接（servlet容器同时也是一个简单的web服务器）
2. 浏览器将相关数据打包（按照http协议要求，创建一个请求数据包），发送给web服务器
3. web服务器拆包（按照http协议要求，将请求数据包的相关数据解析出来），然后将解析到的数据添加到request对象里，同时创建一个response对象。
4. web服务器创建servlet对象，然后调用该对象service方法处理请求（在service方法中可以通过request对象获取请求数据包中的数据，同时将处理结果写到response对象里）
5. web服务器将response对象中的数据取出，打包（按照http协议创建响应数据包），然后发送给浏览器
6. 浏览器拆包，（按http协议要求，将响应数据包中数据解析），然后生成相应的页面。
 <img src= "/assets/files/servlet.jpg" alt="加载错误" title="servlet运行"/>

### 常见错误及解决方式
1. 404：地址写错或没有部署
* 解决方案：先检查是否部署，再检查http://ip:port/appname/url-pattern地址
2. 500：程序运行出错或没有继承HttpServlet或`<servlet-class>`配置的类名出错
* 解决方案：程序代码写严谨，比如对请求参数值作合法性检查；继承HttpServlet；检查类名是否完整正确
3. 405：service没有按照规定编写
* 解决方案：按照service方法的要求编写

### http协议
http协议是一种网络应用层协议，规定了浏览器与web服务器之间如何通信及通信所使用的格式。
1. 如何通信：一次请求，一次连接。利用有线连接为尽可能多的客户请求服务
2. 数据格式
    * 请求数据包：请求行(请求类型(get/post),请求资源路径,协议类型和版本);若干消息头;实体内容;
    * 发送数据包：状态行(协议类型和版本，状态码，状态描述);若干消息头;实体内容:存放程序处理结果
3. 两种请求类型：
    * 直接输入某个地址/点击链接/表单默认提交方式 都是发送get请求,它会将请求参数添加到请求资源路径后，请求行只能存放大约2k数据，只能提交少量数据。参数显示在地址栏，不安全
    * 设置表单的method属性值为post,会将请求参数添加到实体内容中，可提交大量数据。不会将参数显示在浏览器地址栏，相对安全(不会加密)。

### 乱码问题
1. out.prinln默认使用ISO-8859-1编码 
    * 指定out.println方法在输出时使用制定字符集编码 response.setContentType("text/html;charset=utf-8");
2. 表单提交时，服务器默认使用ISO-8859-1解码，产生乱码
    * post解决方法 ：request.setCharacterEncoding("utf-8");
    * get解决方法：在servers.xml中配置 URIEncoding="UTF-8"

### servlet访问数据库
1. 导包：ojdbc，dbcp，（junit）
2. 将config.properties文件添加到resource文件夹下

### servlet的生命周期
#### 什么是servlet的生命周期
容器如何创建servlet对象，如何对其进行初始化，如何调用servlet对象处理请求，以及如何销毁servlet对象的整个过程。
#### 生命周期分成哪几个阶段
1. 实例化；
* 什么时候实例化
    * 默认容器收到请求后，才会创建;容器在默认情况下，只会创建一个实例
    * 容器启动后，立即创建，需要配置：`<load-on-startup>1</load-on-startup>`;
2. 初始化
* 什么是初始化
    * 容器在创建好servlet对象之后，会立即调用该对象的init方法
    * init方法已被GenericServlet实现,会将容器传进来的ServletConfig对象保存下来,并且提供了一个getServletConfig方法来获得ServletConfig对象
    * init方法只会执行一次
    * 可以override init()方法来扩展
    * 初始化参数: 调用ServletConfig提供的getInitParameter方法
3. 就绪(调用)
* *什么是就绪
容器收到请求后，调用servlet对象的service方法来处理请求
* service方法
    * HttpServlet已实现service方法：依据请求类型调用对应的方法(比如，get请求会调用doGet方法)
    * 可以选择override HttpServlet的service方法或者voerride HttpServlet的doGet和doPost方法
4. 销毁
* 什么是销毁
容器在销毁servlet对象之前，会调用该对象的destroy方法。
* destroy方法
GenericeServlet已实现该方法；可以override destroy方法；只会执行一次

#### 生命周期相关类和接口
1. Servlet接口
    * init();初始化
    * service()；实例化
    * destroy()；销毁
2. GenericServlet抽象类
实现了Servlet接口中的init和destroy方法。
3. HttpServvlet抽象类
继承GenericServlet，实现了service方法

### 重定向
服务器通知浏览器向一个新的地址发送请求的，通常服务器发送一个302状态码和一个location消息头（该消息头的值即重定向地址），浏览器收到之后，会立即向重定向地址发送请求
1. 方法：response.sendRedirect(String url)
2. 注意：重定向之前，容器会先清空response对象上存放的所有数据；
3. 特点：重定向地址是任意的;重定向后，浏览器地址栏的地址会发生改变
4. 读取请求参数值
* String request.getParmeter(String name);
    * 如果请求参数名与实际请求参数名不一致，会得到null值；表单提交时若没填写任何数据，会得到空字符串
* String[] request.getParameterValues(String name);
    * 当有多个请求参数名相同时，使用该方法，如 多选框中的值对于多选框，如果一个都不选，则浏览器不会将多选框的值发送给服务器(会获得null值)

### 转发
#### 什么是转发
一个web组件将未完成的任务转交给另外一个web组件继续做。通常是servlet将未完成的处理转发给jsp来展现
#### 如何转发
1. 将一些数据绑定到request对象上: request.setAttribute(String name,Object obj)
    * name是绑定名，obj是绑定值
2. 获得转发器: RequestDispatcher rd=request.getRequestDispatcher(String url);
    * url指的是另外一个web组件的地址（比如某个jsp）
3. 调用转发器的方法来转发: rd.forward(request，response);

#### 转发特点
1. 转发之后，浏览器地址栏的地址不变。
2. 转发的地址有限制（同一个web应用）。

#### 比较转发与重定向
1. 能否共享request和response: 转发可以，重定向不行；
    * 容器收到请求之后，会立即创建request和response，一旦响应发送完毕，则容器会立即销毁这两个对象。
2. 浏览器地址栏地址是否会发生变化: 转发不变，重定向会变。
3. 目的地是否有限制: 转发有限制，要求是同一个应用；重定向没有

### 容器如何处理请求资源路径
比如输入http://ip:port/servlet/xxx.html (/servlet/xxx.html就是请求资源路径)
1. 容器默认认为访问的是一个servlet
2. 容器在web.xml文件当中查找与请求地址匹配的servlet
* 匹配规则
    * 精确匹配  url-pattern的值必须是 /xxx.html
    * 通配符匹配 使用\*表示零个或多个任意的字符 `<url-pattern>/*<url-pattern>`
    * 后缀匹配 使用*.开头，后接任意的多个字符`<url-pattern>*.do<url-pattern>` 匹配所有以.do结尾的请求
3. 如果找不到匹配的servlet,则会查找对应位置的文件，若找到则返回，否则返回404；
4. 路径问题
* 相对路径：不以/开头
* 绝对路径：以/开头
* 如何写绝对路径: 链接地址，表单提交，重定向从应用名开始写；转发从应用名之后开始写
    * 尽量避免直接将应用名写在地址里，而应该使用`String request.getContextPath()`来代替
    * 尽量优先使用绝对路径

### 如何将多个servlet合并为一个
1. 将servlet配置成后缀匹配;`<url-pattern>*.do<url-pattern>`
2. 分析请求资源路径，依据分析结果调用不同的分支来处理。

### 状态管理
将浏览器与web服务器之间多次交互当作一个整体来看待（即为了完成某个业务，需要多次交互，比如购物），并且将多次交互所涉及的数据（即状态）保存下来
#### 如何进行状态管理
1. 客户端状态管理：将状态保存在客户端（浏览器）；常见技术有Cookie
2. 服务器端状态管理：将状态保存在服务器端；常见技术有Session(会话)

#### Cookie
服务器临时存放在浏览器端的少量的数据
1. 工作原理：浏览器访问服务器时，服务器将少量数据以set-cookie消息头的方式发送给浏览器；浏览器会将这些数据临时保存下来，当浏览器再次访问服务器时，会将这些数据以cookie消息头的方式发送给服务器；
2. 添加cookie：`Cookie c=new Cookie(String name,String value);response.addCookie(c);`
3. 查询cookie：`Cookie[] request.getCookies();` 也有可能返回空
4. Cookie的生存时间
    * 默认情况下，浏览器会将cookie保存在内存里。（浏览器不关闭，cookie一直在）
    * 可以调用`cookie.setMaxAge(int seconds)`方法来设置cookie的生存时间.
        * 当seconds>0,浏览器会将cookie保存在硬盘上，超过指定时间，浏览器会销毁该cookie，
   	    * 当seconds<0,保存在内存里面（缺省值）
        * 当seconds=0,浏览器会立即删除该cookie
5. 编码问题
cookie只能保存合法的ASCⅡ字符。需要将中文进行编码处理（将中文转换成相应的ASCⅡ字符串的形式）
`String URLEncoder.encode(String str,String carset)`
`String URLDecoder.decode(String str,String charset)`
6. cookie路径问题
    * 浏览器访问服务器上的某个地址时,会比较该地址是否与cookie的路径匹配,只有匹配的cookie才会被发送
    * 匹配规则：要访问的路径(地址)必须等于cookie的路径或者是其子路径
    * 默认路径：默认路径等于添加该cookie的组件的路径；
        比如：/servlet/app/addCookie.jsp添加了一个cookie，则cookie默认路径是/servlet/app
7. 如何修改cookie的路径: cookie.setPath(String url);
8. cookie的限制
    * cookie是可以被禁止的
    * cookie只能保存少量的数据（大约是4k左右）
    * 浏览器通常只允许保存几百个cookie
    * v对于某一固定站点，只保存20个左右。
    * cookie不安全（若要以cookie方式保存在浏览器，则需加密）

#### Session
服务器端为了保存状态而创建的一个特殊对象
1. 工作原理：浏览器访问服务器时，服务器会创建一个ssession对象，该i对象有一个唯一的id，一般称之为sessionId。服务器会将这个sessionId以cookie的形式发送给浏览器，浏览器会保存下来。当浏览器再次访问服务器时，会将sessionid以cookie的形式发送给服务器，服务器会依据sessionId找到对应的session对象。
2. 获得session对象：`HttpSession s=request.getSession(boolean flag)`
    * 当flag为true时：先查看请求当中是否有sessionId，如果没有则创建session对象，如果有，则依据sessionId查找对应的session对象，找的则返回，若找不到则创建一个新的session对象。
    * 当flag为false时：先查看请求当中是否有sessionId，如果没有则返回null，如果有则依据sessionId查找对应的session对象，找到了则返回，找不到则返回null；
3. 使用session绑定数据
```java
//绑定数据
setAttribute(String name,Object obj)
//依据绑定名获得绑定指
Object  getAttribute(String name)
//解除绑定
removeAttribute(String name)
```
4. session超时
服务器会将空闲时间过长的session对象删除掉.服务器默认的空闲时间一般是半个小时（可修改服务器配置）
`<session-config><session-timeout>30</session-timeout></session-config>`
`setMaxInactiveInterval(int seconds)`

#### 比较session和cookie
1. 优点：session相当于cookie，要安全一些，能保存更丰富的数据类型，能保存更多的数据
2. 缺点：session是将状态保存在服务器端，如果用户量大，会占用大量的内存空间。

### 过滤器
servlet规范当中定义的一种特殊的组件，servlet容器在收到请求之后，会先调用过滤器，再调用servlet。
1. 在java类中实现Filter接口，在doFilter方法中编写处理逻辑，然后在web.xml中配置过滤器。
2. 过滤器的优先级：如果有多个过滤器都满足拦截要求，容器会依据`<filter-mapping>`配置的先后顺序来执行
 <img src= "/assets/files/Filter.jpg" alt="加载错误" title="过滤器"/>

### 监听器
servlet规范当中定义的一种特殊组件，用来监听容器产生的事件并进行处理；
* 容器会产生的两大类事件:
    * 生命周期相关事件：容器创建或销毁了request/session/servlet上下文时产生的事件
    * 绑定数据相关事件：执行了setAttribute/removeAttribute时产生的事件
1. 如何写一个监听器？
* 在java类中实现监听器接口(依据监听的事件类型，选择相应的监听器接口，比如要监听session对象的创建和销毁，需实现HttpSessionListener接口)
* 在监听器接口方法中实现监听器处理逻辑。
* 配置监听器。
3. Servlet上下文
* 容器启动后，会为每一个web应用创建唯一一个符合ServletContext接口要求的对象，该对象一般称之为Servlet上下文
* 该对象特点:
    * 唯一性：一个web应用对应一个servlet上下文；
    * 持久性：容器只要没有关闭或应用没有被卸载则servlet上下文会一直存在
* 如何获得上下文对象：GenericServlet,HttpSession,ServletConfig,FilterConfig提供了getServletContext()来获得上下文
 <img src= "/assets/files/Listener.jpg" alt="加载错误" title="监听器"/>

 4. 作用
* 绑定数据：setAttribute,getAttribute,removeAttribute
* 读取全局的初始化参数：①配置初始化参数②通过ServletContext提供的getInitParameter方法读取

### Servlet线程安全问题
容器在默认情况下，只会创建一个Servlet实例。容器收到请求之后，会启动一个线程来处理请求，就有可能有多个线程同时调用某个servlet(比如都要去修改servlet的某个属性)就有可能产生线程安全问题。通过使用synchronized加锁。

## JSP(java server page)
JSP是sun公司制定的一种服务器端动态页面结束规范。直接使用servlet虽然可以生成动态页面，但是过于繁琐(需要使用out.println语句输出),不利于页面维护，故制定了jsp技术规范。
### 如何写一个jsp
jsp是一个以.jsp为后缀的文件，该文件的内容主要是html和少量的java代码。该文件会被容器转换成一个servlet然后执行。
1. 写一个以.jsp为后缀的文件；
2. 在该文件当中，可以添加如下内容：
* html(css,js)
* java代码
    * java代码片段： <%java代码片段%>
    * jsp表达式：<%=java表达式%>
    * jsp声明：    <%!   %>添加新的属性或方法
* 隐含对象
    * 定义:在jsp文件里面，可以直接使用的对象，比如，out,request,response.
    * 为什么可以使用这些隐含对象: 因为容器会自动添加获得这些对象的语句。
* 指令
    * 告诉容器，在讲jsp文件转换成java文件时，做一些额外处理，比如导包
    * 语法: <%@ 指令名   属性=值 %>
    * page指令
        * import属性:导包,比如`<%@ import="java.util.Date,java.util.text%>`
        * contentType属性：设置response.setContentTye方法的参数值
        * pageEncoding属性：告诉容器，在读取jsp文件内容时，使用指定字符集解码
        * session属性：缺省值为true，如果值为false，session隐含对象就不能用了
        * errorPage属性：用来指定一个异常处理页面
        * isErrorPage属性：缺省值false，如果值为true表示这是一个异常处理页面，可以使用exception隐含对象了
    * include指令
        * file属性:容器在将jsp文件转换成java文件时将file属性所指定的文件内容插入该指令所在位置
    * taglib指令:导入jsp标签
* 注释
    * `<!--注释内容-->` 注释内容如果是java代码，会执行，但不会输出结果
    * `<%--注释内容--%>`注释内容如果是java代码，不会执行

### jsp是如何运行的？
1. 容器将.jsp文件先转换成一个.java文件(也就是一个servlet); html会在service方法里,使用out,write()输出;而java代码片段会在service方法里照原样输出;jsp表达式会在service方法里使用out.print输出
2. 容器调用该servlet来处理请求。

### jsp标签和el表达式
#### jsp标签是什么
jsp标签语法类似html标签，用来替换jsp文件中的java代码。因为直接在jsp文件中写java代码不利于jsp文件的维护。
#### jsp标签的用法
jstl标签(java standard tag lib):apache开发的一套jsp标签，捐献给sun，sun将其命名为jstl
1. 核心标签：使用taglib指令引入要使用的标签`<%@taglib uri="" prefix=""%>`
    * if标签：`<c:if test="" var="" scope=""></c:if>` 当test属性值为true时执行标签体内容
        * test属性:可以使用el表达式进行计算；
        * var属性:指定一个绑定名
        * scope属性:指定绑定的范围(“page”,"requset”,“session”,"application"）
    * choose标签:
    ```
    <c:choose>
        <c:when test=""></c:when>(可以出现一次或多次)when表示一个分支,test属性值为true时会执行标签体内容
        <c:otherwise></c:otherwise>(可以出现0次或一次)otherwise表示例外，当when都为false是执行
    </c:choose>
    ```
    * forEach标签:`<c:forEach items="" var="" varStatus=""></c:forEach>`
        * items属性：用来指定要遍历的集合或数组，可以使用el表达式
        * var属性：指定绑定名，绑定范围是pageContext  
        * varStatus属性：指定绑定名，绑定范围是pageContext‘
        * getIndex()：获得正在被遍历的元素的下标（从0开始）
        * getCount():获得当前正在被遍历的元素是第几个元素(从1开始)
    * 自定义标签：
        * 写一个java类继承SimpleTagSupport类；
        * 在doTag方法中，编写处理逻辑
        * 在tld文件中描述标签

#### el表达式是什么
一套简单的运算规则，用于给jsp标签的属性赋值，也可以直接输出。

#### 使用el表达式 
1. 访问bean属性(bean是一个java类，public,有无参构造器，有一些属性以对应的get/set方法)
* 方式1：`${user.name}` 容器会依次从pageContext，request，session，application当中取绑定名为user的对象，然后调用该对象的getName方法，最后输出。会将null转换成“”输出；如果依据绑定名找不到对应的对象，则会输出“”;可以使用pageScope/requestScope/sessionScope/applicationScope来指定查找范围
* 方式2：`${user['name']}`等价于方式1
* 特殊用法：[]里可以出现绑定名; []里可以出现从0开始的下标(访问数组某元素);进行一些简单的计算,计算结果可以直接输出，也可以用来给jsp标签的属性赋值。
    * 算数运算：+ - * / %  ； +只能求和，
    * 关系运算：> <  >=  <=  ==  !=
    * 逻辑运算： && \|\| ！
    * empty运算：集合内容是否为空或者是否为一个空字符串
* 获取请求参数值
    * ${param.name}等价于request.getParameter("name")
    * ${paramValues.interest}等价于request.getParameterValues("interest")