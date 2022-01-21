# Java面试题



### 一.  JAVA基础

1. 说说List,Set,Map三者的区别？
List(对付顺序的好帮手)： List接口存储一组不唯一（可以有多个元素引用相同的对象），有序的对象
Set(注重独一无二的性质): 不允许重复的集合。不会有多个元素引用相同的对象。
Map(用Key来搜索的专家): 使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象

2. HashMap 和 Hashtable 的区别
线程是否安全： HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过synchronized 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；
效率： 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；
对Null key 和Null value的支持： HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛出 NullPointerException。
初始容量大小和每次扩充容量大小的不同 ： ①创建时如果不指定容量初始值，Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小（HashMap 中的tableSizeFor()方法保证，下面给出了源代码）。也就是说 HashMap 总是使用2的幂作为哈希表的大小,后面会介绍到为什么是2的幂次方。
底层数据结构： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

3. comparable 和 Comparator的区别
comparable接口实际上是出自java.lang包 它有一个 compareTo(Object obj)方法用来排序
comparator接口实际上是出自 java.util 包它有一个compare(Object obj1, Object obj2)方法用来排序
一般我们需要对一个集合使用自定义排序时，我们就要重写compareTo()方法或compare()方法，当我们需要对某一个集合实现两种排序方式，比如一个song对象中的歌名和歌手名分别采用一种排序方法的话，我们可以重写compareTo()方法和使用自制的Comparator方法或者以两个Comparator来实现歌名排序和歌星名排序，第二种代表我们只能使用两个参数版的 Collections.sort().

4. System.out.println(3|9)输出什么?
正确答案：11。
考察知识点：&和&&；|和||
&和&&：
共同点：两者都可做逻辑运算符。它们都表示运算符的两边都是true时，结果为true；
不同点: &也是位运算符。& 表示在运算时两边都会计算，然后再判断；&&表示先运算符号左边的东西，然后判断是否为true，是true就继续运算右边的然后判断并输出，是false就停下来直接输出不会再运行后面的东西。
|和||：
共同点：两者都可做逻辑运算符。它们都表示运算符的两边任意一边为true，结果为true，两边都不是true，结果就为false；
不同点：|也是位运算符。| 表示两边都会运算，然后再判断结果；|| 表示先运算符号左边的东西，然后判断是否为true，是true就停下来直接输出不会再运行后面的东西，是false就继续运算右边的然后判断并输出。
回到本题：
3 | 9=0011（二进制） | 1001（二进制）=1011（二进制）=11（十进制）



### 二. J2EE

1. Cookie和Session的的区别?
Cookie 和 Session都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。
Cookie 一般用来保存用户信息 比如①我们在 Cookie 中保存已经登录过得用户信息，下次访问网站的时候页面可以自动帮你登录的一些基本信息给填了；②一般的网站都会有保持登录也就是说下次你再访问网站的时候就不需要重新登录了，这是因为用户登录的时候我们可以存放了一个 Token 在 Cookie 中，下次登录的时候只需要根据 Token 值来查找用户即可(为了安全考虑，重新登录一般要将 Token 重写)；③登录一次网站后访问网站其他页面不需要重新登录。Session 的主要作用就是通过服务端记录用户的状态。 典型的场景是购物车，当你要添加商品到购物车的时候，系统不知道是哪个用户操作的，因为 HTTP 协议是无状态的。服务端给特定的用户创建特定的 Session 之后就可以标识这个用户并且跟踪这个用户了。
Cookie 数据保存在客户端(浏览器端)，Session 数据保存在服务器端。
Cookie 存储在客户端中，而Session存储在服务器上，相对来说 Session 安全性更高。如果使用 Cookie 的一些敏感信息不要写入 Cookie 中，最好能将 Cookie 信息加密然后使用到的时候再去服务器端解密。

2. JSP中的四种作用域?
JSP中的四种作用域包括page、request、session和application，具体来说：
page代表与一个页面相关的对象和属性。
request代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。
session代表与某个用户与服务器建立的一次会话相关的对象和属性。跟某个用户相关的数据应该放在用户自己的session中。
application代表与整个Web应用程序相关的对象和属性，它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。

3. request.getAttribute()和 request.getParameter()有何区别?
从获取方向来看：
getParameter()是获取 POST/GET 传递的参数值；
getAttribute()是获取对象容器中的数据值；
从用途来看：
getParameter()用于客户端重定向时，即点击了链接或提交按扭时传值用，即用于在用表单或url重定向传值时接收数据用。
getAttribute() 用于服务器端重定向时，即在 sevlet 中使用了 forward 函数,或 struts 中使用了 mapping.findForward。 getAttribute 只能收到程序用 setAttribute 传过来的值。
另外，可以用 setAttribute(),getAttribute() 发送接收对象.而 getParameter() 显然只能传字符串。 setAttribute() 是应用服务器把这个对象放在该页面所对应的一块内存中去，当你的页面服务器重定向到另一个页面时，应用服务器会把这块内存拷贝另一个页面所对应的内存中。这样getAttribute()就能取得你所设下的值，当然这种方法可以传对象。session也一样，只是对象在内存中的生命周期不一样而已。getParameter()只是应用服务器在分析你送上来的 request页面的文本时，取得你设在表单或 url 重定向时的值。
总结：
getParameter()返回的是String,用于读取提交的表单中的值;（获取之后会根据实际需要转换为自己需要的相应类型，比如整型，日期类型啊等等）
getAttribute()返回的是Object，需进行转换,可用setAttribute()设置成任意对象，使用很灵活，可随时用

4. JSP有哪些内置对象、作用分别是什么
JSP有9个内置对象：
request：封装客户端的请求，其中包含来自GET或POST请求的参数；
response：封装服务器对客户端的响应；
pageContext：通过该对象可以获取其他对象；
session：封装用户会话的对象；
application：封装服务器运行环境的对象；
out：输出服务器响应的输出流对象；
config：Web应用的配置对象；
page：JSP页面本身（相当于Java程序中的this）
exception：封装页面抛出异常的对象。

5. get和post请求的区别
网上也有文章说：get和post请求实际上是没有区别，大家可以自行查询相关文章（参考文章：https://www.cnblogs.com/logsharing/p/8448446.html，知乎对应的问题链接：get和post区别？）！我下面给出的只是一种常见的答案。
①get请求用来从服务器上获得资源，而post是用来向服务器提交数据；
②get将表单中数据按照name=value的形式，添加到action 所指向的URL 后面，并且两者使用"?"连接，而各个变量之间使用"&"连接；post是将表单中的数据放在HTTP协议的请求头或消息体中，传递到action所指向URL；
③get传输的数据要受到URL长度限制（最大长度是 2048 个字符）；而post可以传输大量的数据，上传文件通常要使用post方式；
④使用get时参数会显示在地址栏上，如果这些数据不是敏感数据，那么可以使用get；对于敏感数据还是应用使用post；
⑤get使用MIME类型application/x-www-form-urlencoded的URL编码（也叫百分号编码）文本的格式传递参数，保证被传送的参数由遵循规范的文本组成，例如一个空格的编码是"%20"。
补充：GET方式提交表单的典型应用是搜索引擎。GET方式就是被设计为查询用的。

6. 说一下转发(Forward)和重定向(Redirect)的区别
转发是服务器行为，重定向是客户端行为。
转发（Forword） 通过RequestDispatcher对象的forward（HttpServletRequest request,HttpServletResponse response）方法实现的。RequestDispatcher 可以通过HttpServletRequest 的 getRequestDispatcher()方法获得。例如下面的代码就是跳转到 login_success.jsp 页面。
request.getRequestDispatcher("login_success.jsp").forward(request, response);
重定向（Redirect） 是利用服务器返回的状态吗来实现的。客户端浏览器请求服务器的时候，服务器会返回一个状态码。服务器通过HttpServletRequestResponse的setStatus(int status)方法设置状态码。如果服务器返回301或者302，则浏览器会到新的网址重新请求该资源。
从地址栏显示来说：forward是服务器请求资源，服务器直接访问目标地址的URL，把那个URL的响应内容读取过来，然后把这些内容再发给浏览器。浏览器根本不知道服务器发送的内容从哪里来的，所以它的地址栏还是原来的地址。redirect是服务端根据逻辑，发送一个状态码，告诉浏览器重新去请求那个地址。所以地址栏显示的是新的URL。
从数据共享来说：forward：转发页面和转发到的页面可以共享request里面的数据。redirect：不能共享数据。
从运用地方来说：forward：一般用于用户登陆的时候，根据角色转发到相应的模块。redirect：一般用于用户注销登陆时返回主页面和跳转到其它的网站等。
从效率来说：forward：高。redirect：低。





### 三. 框架使用

1. 什么是 Spring 框架?
Spring 是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性。
我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块。比如：Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。
Spring 官网列出的 Spring 的 6 个特征:
核心技术 ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
测试 ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
数据访问 ：事务，DAO支持，JDBC，ORM，编组XML。
Web支持 : Spring MVC和Spring WebFlux Web框架。
集成 ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
语言 ：Kotlin，Groovy，动态语言。

2. Spring 中的 bean 的作用域有哪些?
singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
prototype : 每次请求都会创建一个新的 bean 实例。
request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。

3. Spring AOP IOC 实现原理
IOC： 控制反转也叫依赖注入。IOC利用java反射机制，AOP利用代理模式。IOC 概念看似很抽象，但是很容易理解。说简单点就是将对象交给容器管理，你只需要在spring配置文件中配置对应的bean以及设置相关的属性，让spring容器来生成类的实例对象以及管理对象。在spring容器启动的时候，spring会把你在配置文件中配置的bean都初始化好，然后在你需要调用的时候，就把它已经初始化好的那些bean分配给你需要调用这些bean的类。
AOP： 面向切面编程。（Aspect-Oriented Programming） 。AOP可以说是对OOP的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码，属于静态代理。