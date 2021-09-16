## 1.Web服务器和Web应用

​	**Web服务器**：Web服务器是由专门的服务器开发商创建的，具备动态执行第三方创建的Web应用中的特定程序代码的能力

​	**Web应用**：Web应用是由专门的应用软件开发商创建的，包含了能处理特定业务的程序代码。



​	**中介方（Servlet接口）**：为了能让Web服务器与Web应用这两个不同的软件系统协作，需要一个中介方指定Web应用与Web服务器进行协作的标准接口，Servlet是最主要的一个接口。

​	Servlet接口规定：

​		1.Web服务器可以访问任意一个Web应用中所有实现Servlet接口的类

​		2.Web应用中用于被Web服务器动态调用的程序代码位于Servlet接口的实现类中

![image-20210530192228949](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210530192228949.png)

**Servlet规范**：Oracle公司规定了一些系的Web应用与Web服务器之间的Java接口，这些接口统称为Servlet规范。

​	Servlet规范规定：

​		能够发布和运行Java Web应用的Web服务器称为**Servlet容器**。Servlet容器最主要的特征是动态执行Java Web应用(基于Java的Web应用)中Servlet实现类的程序代码。简言之，Servlet容器是一种Web服务器。

​	Tomcat是一种符合Servlet规范的Servlet容器，即一种Web服务器。**准确的说Tomcat包含Servlet容器，因为Tomcat还有其他组件，而Servlet容器是Tomcat的核心组件**



## 2. Tomcat的基本功能

​	**(Java)Servlet**：运行在服务器上的程序，作为来自客户端的请求和服务器上的数据库或应用程序（某个业务）之间的中间层。可以理解为一个Servlet用于处理客户端的一个特定请求。

​	Servlet容器响应客户请求访问特定Servlet的流程：

​		![image-20210530195019767](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210530195019767.png)

​	（1）：客户发出要求访问特定Servlet的请求

​	（2）：Servlet容器接受到客户请求，对其进行解析

​	（3）：Servlet容器创建一个ServletRequest对象，在ServletRequest对象中包含了客户请求信息以及其他关于客户的相关信息，如请求头，请求正文，以及客户机的IP地址等等

​	（4）：Servlet容器创建一个ServletResponse对象

​	（5）：Servlet容器调用客户所请求的Servlet的service()服务方法，并把ServletRequest和ServletResponse对象作为参数传给该service方法

​	（6）：Servlet从ServletRequest对象中可获得客户的请求信息

​	（7）：Servlet利用ServletResponse对象来生成响应结果

​	（8）：Servlet容器把Servlet生成的响应结果发送给客户



## 3.Tomcat的组成结构

​	这一部分需要找个视频看了之后补一补



## 4.Tomcat的目录结构

​	bin ——Tomcat执行脚本目录
​	conf ——Tomcat配置文件
​	lib ——Tomcat运行需要的库文件（JARS）
​	logs ——Tomcat执行时的LOG文件

​	temp ——Tomcat临时文件存放目录
​	webapps ——Tomcat的主要Web发布目录（存放我们自己的JSP,SERVLET,类）
​	work ——Tomcat的工作目录，Tomcat将翻译JSP文件得到的Java文件和class文件放在这里。

![image-20210530214725908](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210530214725908.png)



**Tomcat下的lib目录和webapps中WEB-INF/lib目录的区别**：

​	Tomcat的lib子目录：存放的JAR文件不仅能被Tomcat访问，还能被所有在Tomcat中发布的Java Web应用访问

​	Java Web应用的lib子目录：存放的JAR文件只能被当前Java Web应用访问