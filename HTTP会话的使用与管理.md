当你使用淘宝的时候，你将一个商品加入了你的购物车，服务器怎么从那么多用户的购物车中找到你的购物车？这需要服务器识别出你的身份，然后找到你的购物车，最后把商品放到这个购物车里面。

Web服务器跟踪客户的状态通常有4种方法：

​	1.在HTML表单种加入隐藏字段

​	2.重写URL，使它包含用于跟踪客户状态的数据

​	3.用Cookie来传送用于跟踪客户状态的数据

​	4.使用会话（Session）机制



### 9.1会话简介

​	在Web开发领域，会话机制是用于跟踪客户状态的**普片解决方案**（不局限于java web应用中，在asp,php或者jsp开发的web应用中同样适用）。

​	**会话：在一段时间内，单个客户与Web应用的一连串相关的交互过程**

​	在一个会话中，客户可能会**多次请求访问Web应用的同一个网页**，也有可能**请求访问同一个Web应用中的多个网页**

​		例如：在电子邮件系统中，从一个客户登录到电子邮件系统开始，经过收信、写信和发信直至推出登录系统，整个过程是一个会话



​	**会话运作流程**

​		1.一个浏览器在第一次请求Web应用中支持会话的网页的时候，Servlet容器试图寻找HTTP请求中表示Session ID的Cookie，由于还不存在这样的Cookie，Servlet容器会创建一个HttpSession对象，为它分配文艺的Session ID,然后把Session ID作为Cookie添加到HTTP响应结果中，浏览器接收到响应结果后，会把表示Session ID的Cookie保存在客户端

​		2.该浏览器进程继续请求访问Web应用中任意一个支持会话的网页，在本次请求中会包含表示Session ID的Cookie，Servlet容器试图寻找HTTP请求中表示Session ID的Cookie,由于此时能获得这样的Cookie，因此认为本次请求已经处于一个会话中了，Servlet容器从Cookie中获取Session ID,然后根据Session ID**找到内存中对应的HttpSession对象**

​		3.浏览器重复步骤2，直至当前**会话被销毁**（会话被销毁的时机请参考9.2），从而内存中对应的HttpSession就会结束生命周期。

**在一个会话中，将表示会话的Cookie setMaxAge(-1),表示Cookie的有效期为-1，意味着该Cookie仅仅存在于当前浏览器进程中，当浏览器进程关闭，Cookie也就消失**



### 9.2 HttpSession的声明周期及会话范围

​	会话范围指浏览器端与一个Web应用进行一次会话的过程，在具体实现上，会话范围与HttpSession对象的生命周期对应，因此想要在一个会话中共享数据，通过HttpSession对象就能实现。

​	**HttpSesssion接口中与共享数据相关的方法：**

```java
//Returns the object bound with the specified name in this session, or null if no object is bound under the name.
getAttribute(java.lang.String name)
//Returns an Enumeration of String objects containing the names of all the objects bound to this session.
getAttributeNames()	
//Binds an object to this session, using the name specified.
setAttribute(java.lang.String name, java.lang.Object value)	
//Removes the object bound with the specified name from this session.
removeAttribute(java.lang.String name)
//Invalidates this session then unbinds any objects bound to it.
invalidate()
//Specifies the time, in seconds, between client requests before the servlet container will invalidate this session. interval <  0 insists that this session will never expire
setMaxInactiveInterval(int interval)
```



**HttpSession的创建时机**（即一个新的会话开始）（以下情况满足其一即可）：

​	1.一个浏览器进程第一次访问Web应用中支持会话的任意一个网页

​	2.浏览器进程与Web应用的一次会话已经被销毁后，浏览器进程再次访问Web应用中支持会话的任意一个网页



**HttpSession结束声明 周期的时机**（即一个会话的销毁）（以下情况满足其一即可）：

​	1.浏览器进程终止（关闭了浏览器）

​	2.服务器端执行HttpSession对象的invalidate()方法

​	3.会话过期（指会话开始后，如果一段时间内客户一直没有和Web应用交互，那么Servlet容器会自动销毁这个会话，HttpSession类的setMaxInactiveInterval(int interval)方法用于设置允许会话保持不活动状态的时间，单位为秒）

**Tomat为会话设定的默认保持不活动状态时间为1800s**

**Tomcat中的Web应用被终止时，它的会话不会被销毁，而是被Tomcat持久化到永久性存储设备中，当Weeb应用被重启后，Tomcat会重新加载这些会话（详情参考9.5会话的持久化）**



### 9.3 在Servlet中使用会话

​	JSP文件默认情况下都支持会话，而HttpServlet类默认情况下不支持会话.

​	在JSP文件中，可以直接通过固定变量session来引用隐含的HttpSession对象

​	**在Servlet中需要通过HttpServletRequest对象来获得HttpSession对象**：

​		getSession(): 如果存在会话，则返回会话，否则创建一个新的会话（HttpSession对象）并返回，此时该方法相当于getSession(true)

​		getSession(boolean create): 如果create为true，则创建一个会话并返回，如果create为false，则返回已经存在的HttpSession对象，如果此时不存在HttpSession对象，则返回一个null



### 9.4 通过重写URL来跟踪会话（解决浏览器禁用Cookie后维持会话的方法）

​	9.1中讲过，Servlet容器先在客户端浏览器中保存一个Session ID,之后浏览器发出的HTTP请求中就会包含这个Session ID. Servlet容器读取HTTP请求中的Session ID,就能判断出来自各个浏览器进程的HTTP请求属于哪一个会话，这一过程也称为会话的跟踪。

​	如果浏览器支持Cookie，Servlet容器就能把Session ID 作为Cookie保存在浏览器中，**但是如果浏览器出于安全原因禁用Cookie，不允许服务器向客户端存放Cookie,那么Servlet容器跟踪会话就需要使用另一种方式——重写URL**

​	**重写URL**：把Session ID添加到URL信息中，HttpServletResponse接口提供了重写URL的方法

​	public java.lang.String encodeURL(String url)

​	该方法的运行流程如下;

​		1. 先判断请求的URL所对应的web服务是否支持会话，如果不支持，那么直接返回该URL

​		2.再判断浏览器是否支持Cookie，如果支持Cookie,就直接返回该URL； 如果浏览器不支持Cookie,就在参数指定的URL中加入当前Session ID的信息，然后返回修改后的URL

​	**由此可见，只有当Web组件支持会话，并且浏览器不支持Cookie的情况下，encodeURL方法才会重写URL，否则直接返回原来的URL**



### 9.5 会话的持久化

​	当一个会话开始时，Servlet容器会为会话创建一个HttpSession对象。Servlet容器在**某些情况下**会把这些HttpSession对象从内存中转移到永久性存储设备(文件系统或者数据库)中，在需要访问HttpSession信息时再把他们加载到内存中。

​	把内粗中的HttpSession对象保存到文件系统或者数据库中，这一过程称为会话的持久化，会话的持久化有两个好处：

​		1.节约内存空间：当很多客户端在同一个时间段内访问服务器的时候，会产生大量HttpSession对象，占用了服务器端大量的内存资源，将HttpSession对象持久化，当需要使用该对象时再从持久层拿出来，这样可以节约内存空间

​		2.确保服务器重启或者单个Web应用重启后，能恢复重启前的会话。

​	**会话保存到持久层中使用的是java语言提供的对象序列化技术，把会话恢复到内存中使用的是java语言中的反序列化技术**



​	**会话持久化和重新加载到内存中的时机**

​		会话持久化的时机：

​			1.服务器终止或单个Web应用终止时，Web应用中的会话被搁置（搁置=持久化）

​			2.会话处于不活动状态的时间太长，达到了特定的限制值

​			3.Web应用中处于运行时状态的会话数太多，达到了特定的阈值，部分会话会被搁置

​		会话重新加载到内存中的时机：

​			1.服务器重启或单个Web应用重启时，Web应用中的会话被激活（激活=重新将会话加载到内存中）

​			2.处于会话中的客户端向Web应用发出HTTP请求，相应的会话被激活

​	

​	Java Servlet API并没有为会话的持久化提供标准的接口，**会话的持久化完全依赖于Servlet容器的具体实现**。
