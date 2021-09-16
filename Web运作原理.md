## 1.1 Web的概念

​	**什么是Web**

​		Web是一种分布式应用架构，旨在共享分布在网络上的各个Web服务器中所有互相连接的信息。

​	**Web的使用的技术**

​		HTML来表达信息

​		URL来实现网络上信息的定位

​		HTTP作为传输协议



## 1.2 HTTP简介

​		请参考w3cschool



## 1.3 URL简介

​		URL一般由以下三个部分组成：

​			**应用层协议**

​			**主机IP地址或者域名**

​			**资源所在路径/文件名**

​		例如：http://www.javathinker.net/JavaWeb/javabook.htm

​		其中，	“http”	是应用层协议

​						“www.javathinker.net"	是Web服务器的域名

​						”JavaWeb"	是文件所在路径

​						“javabook.htm”	是文件名

​		**URL和URI的区别**

URI = Uniform Resource Identifier 统一资源标志符
URL = Uniform Resource Locator 统一资源**定位符**
URN = Uniform Resource Name 统一资源名称

大白话，就是URI是抽象的定义，不管用什么方法表示，只要能定位一个资源，就叫URI，本来设想的的使用两种方法定位：1，URL，用地址定位；2，URN 用名称定位。



## 1.4 Http简介

### 	1.4.1 Http请求格式

​			Http请求格式由如下三部分构成：

​				请求方法、URI和HTTP版本

​				请求头	(Request Header)

​				请求正文（Request Content）

​			示例：

`POST /sample.jsp HTTP/1.1`																	
`Accept:image/gif.image/jpeg,*/*`
`Accept-Language:zh-cn`
`Connection:Keep-Alive`
`Host:localhost`
`User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)`
`Accept-Encoding:gzip,deflate`

`username=jinqiao&password=1234`

第一部分：

`POST	/sample.jsp HTTP/1.1`																	//请求方法：POST	URI：sample.jsp	HTTP协议版本：1.1

第二部分：请求头

`Accept:image/gif.image/jpeg,*/*`
`Accept-Language:zh-cn`
`Connection:Keep-Alive`
`Host:localhost`
`User-Agent:Mozila/4.0(compatible;MSIE5.01;Window NT5.0)`
`Accept-Encoding:gzip,deflate`

请求头包含许多有关客户端环境和请求正文有用的信息，如浏览器类型，所用的语言，正文类型以及正文长度等

第三部分：请求正文（注意和第二部分空了一行）

username=jinqiao&password=1234



### 1.4.2 HTTP响应格式

HTTP响应也由3个部分构成，分别是：

- 协议/版本 状态码 描述
- 响应头(Response Header)
- 响应正文

下面是一个HTTP响应的例子：
`HTTP/1.1 200 OK`
`Server:Apache Tomcat/5.0.12`
`Date:Mon,6Oct2003 13:23:42 GMT`
`Content-Length:112`
`Content-Type:text/html`
`Last-Moified:Mon,6 Oct 2003 13:23:42 GMT`
`Content-Length:112`

`<html>`

　　<head>
　　<title>HTTP响应示例<title>
　　</head>
　　`<body>`
　　　　`Hello HTTP!`
　　`</body>`
`</html>`

**（1）协议/版本 状态码 描述**
协议/版本 状态码 描述：HTTP响应的第一行类似于HTTP请求的第一行，它表示通信所用的协议是HTTP1.1，服务器已经成功的处理了客户端发出的请求（200表示成功）:
	HTTP/1.1 200 OK

常见的状态码：

​		200 ：成功

​		400： 错误的请求，客户发送的HTTP请求不正确

​		404：文件不存在，在服务器上没有客户要求访问的文档

​		405： 服务器不支持客户的请求方式

​		500： 服务器内部错误

**（2）响应头**

(Response Header)响应头也和请求头一样包含许多有用的信息，例如服务器类型、日期时间、内容类型和长度等：
`Server:Apache Tomcat/5.0.12`				//服务器版本号
`Date:Mon,6Oct2003 13:23:42 GMT`
`Content-Length:112`								//响应体长度
`Content-Type:text/html`						//响应正文类型
`Last-Moified:Mon,6 Oct 2003 13:23:42 GMT`
`Content-Length:112`

**响应正文类型有哪些**

![image-20210519204648302](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210519204648302.png)

**（3）响应正文**（响应头和正文之间也必须用空行分隔）
`<html>`

<head>
<title>HTTP响应示例<title>
</head>
`<body>`
`Hello HTTP!`
`</body>`
`</html>`
