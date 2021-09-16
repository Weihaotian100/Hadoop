## 1. 比较HTML、Servlet、JSP

​	HTML: 每次客户端请求访问该类型文件，客户端得到的都是该文件本身同样的内容，HTML标记能够简洁又直观地表达网页的外观，编写起来简单，但其本身是静态的。

​	Servlet: 动态生成HTML页面，动态值根据客户端传入的参数的不同，生成HTML页面内容也会有所不同，Servlet方式中，开发人员必须以编写Java程序代码的方式来生成HTML文档，更确切地说，需要通过PrintWriter对象来一行行地打印HTML文档内容，这个过程是非常繁琐的，尤其对于复杂的大型网页

​	JSP：同时具有以上两者的优点，并摒弃了以上两者的缺点。在传统HTML文件中，加入Java程序片段和JSP标记，就构成了JSP文件

当客户端请求的url对于服务器端的某个JSP文件时，如果该JSP文件首次被请求，那么Web容器就会将该JSP文件翻译为Servlet源文件，再将其编译成Servlet类，接着调用该Servlet类的服务方法。如果该JSP文件不是被首次请求，那么对应的Servlet类已经存在，Web容器会直接调用该Servlet类的服务方法。



**JSP本质上是Servlet,Servlet的功能和特性也全都适用于JSP，因此JSP文件可以访问Servlet API中的接口和类，此外还可以访问JSP API中的接口和类**

​	JSP中的Java程序片段可以完成与Servlet同样的功能，例如动态生成网页、操纵数据库、把请求转发给其他Web组件，以及发送E-Mail等。

**JSP的出现，使得把Web应用中的HTML文档也业务逻辑代码有效分离称为可能，通常JSP负责动态生成HTML文档，而业务逻辑由其他可重用的组件，如JavaBean或者其他Java程序来实现，JSP可以通过Java程序片段来访问这些业务组件。**





## 2. JSP语法

​	JSP语法特点：**尽可能地用标记取代Java程序代码，使整个JSP文件在形式上不想Java程序，而像标记文档**

​	JSP文件中除了可以直接包哦含HTML文本，还可以包含以下内容：

​		JSP指令（或称为指示语句）

​		JSP声明

​		Java程序片段

​		Java表达式

​		JSP隐含对象



### 2.1 JSP指令（Directive）



​	JSP指令的一般语法形式：	**<%@ 指令名 属性=“值”%>**

​	常用的三种指令为：**page、include、taglib**



#### 	1. page指令

​	