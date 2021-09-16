## 1. Java Web应用简介

​	Java Web应用由一组Servlet/JSP、HTML文件、相关Java类，以及其他可以被绑定的资源构成，它可以在由各种供应商提供的符合Servlet规范的Servlet容器中运行。

​	**Servlet**：标准Servlet接口的实现类，运行在服务器端，包含了被Servlet容器动态调用的程序代码

​	**JSP**：包含Java代码的HTML文档，运行在服务器端，当客户端请求访问JSP文件时，Servlet容器先把它编译成Servlet类，然后动态调用生成的Servlet类的程序代码（实质上调用的程序代码是Servlet类的service方法）

​	**相关的Java类**：Web应用开发人员自定义的与Web应用相关的Java类

​	**静态文档**：存放在服务器端的文件系统，如HTML文件、图片文件和声音文件等，当客户端请求这些文件的时候，Servlet容器从本地文件系统中读取这些文件的数据，再把它发送到客户端。

​	**客户端脚本程序**：由客户端来运行的程序，如JavaScript

​	**web.xml文件**：Java web应用的配置文件，**该文件必须位语WEB-INF子目录下**



## 2.Java Web应用的目录结构

​	![image-20210530205723549](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210530205723549.png)

Assembly Root: Web应用的根目录，下面存放web pages文件，如jsp、html

Assembly Root/WEB-INF: 存放web.xml配置文件，**WEB-INF以及WEB-INF下的子目录中的内容不能被浏览器直接访问，只能被Tomcat服务器访问**

Assembly Root/WEB-INF/lib : 存放jar包、类库文件

Assembly Root/WEB-INF/classes ：存放开发人员编写的源文件编译之后的class文件



## 3. Web应用发布到Tomcat服务器上

​	1.把web应用打包成war文件，放到Tomcat根目录的webapp下	

​	2.直接把web应用的根目录整个copy到webapp下



​	通过 bin/startup.bat 启动tomcat服务器，然后在浏览器中输入localhost:8080/web应用根目录名称



## 4. Servlet映射URL

​	客户端用户请求web应用的某个服务，一个服务对应一个Servlet，即请求服务的过程等于用户通过URL访问Servlet的过程，因此Servlet与URL之间需要简历映射关系。

​	**一个Servlet可以映射多个URL，只需要写多个servlet-mapping即可，JSP同样也可以映射URL，JSP的配置参考JSP那一章**

​	Servlet映射URL的方式有两种：

  1. 在web.xml文件中映射URL

     Servlet映射的URL由web.xml文件中的 ***<url-pattern>*** 元素来指定

     例：

     ```xml
     <servlet>
         <servlet-name>dispatcher</servlet-nameservlet-name>
         <servlet-class>mypack.DispatcherServlet</servlet-classs>
     </servlet>
     
     <servlet-mapping>
         <servlet-name>dispatcher</servlet-name>
         <url-pattern>/dispatcher</url-pattern>
     </servlet-mapping>
     ```

     在以上代码中， url (/dispatcher) 与 mypack.DispatcherServlet之间形成了映射关系

     

  2. 用@WebServlet注解来映射URL
