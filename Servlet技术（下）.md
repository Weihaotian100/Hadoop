## Servlet技术（下）

### 5.1下载文件

​	下载文件的实现：把需要下载的文件通过java InputStream 读入转换成二进制文件，再将InputStream中的内容读入到一个byte数组中，最后将byte数组中的内容写入到OutputStream输出流中，OutputStream的实例化对象由Tomcat容器创建的ServletResponse对象的获得

```java
package mypack;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;

public class DownloadServlet extends HttpServlet {
  public void doGet(HttpServletRequest request,HttpServletResponse response)
         throws ServletException, IOException {
    OutputStream out; //输出响应正文的输出流
    InputStream in;  //读取本地文件的输入流
    //获得filename请求参数 
    String filename=request.getParameter("filename");
     
    if(filename==null){
      out=response.getOutputStream(); 
      out.write("Please input filename.".getBytes());
      out.close();
      return;
    }
    
    //创建读取本地文件的输入流
    /* getResourceAsStream(java.lang.String path)
    ** Returns the resource located at the named path as an InputStream object.
    **/
    in= getServletContext().getResourceAsStream("/store/"+filename);
     /* available()
     ** Returns an estimate of the number of bytes that can be read (or skipped over) from this input stream 
     */
    int length=in.available();
    //设置响应正文的MIME类型,MIME类型设置为force-download，浏览器会以下载的方式来处理响应正文
    response.setContentType("application/force-download"); 
    response.setHeader("Content-Length",String.valueOf(length));
    response.setHeader("Content-Disposition", "attachment;filename=\""+filename +"\" "); 
    
    /** 把本地文件中的数据发送给客户 */
    out=response.getOutputStream(); 
    int bytesRead = 0;
    byte[] buffer = new byte[512];
    /* read()
    ** Reads some number of bytes from the input stream and stores them into the buffer array b
    */
    while ((bytesRead = in.read(buffer)) != -1){
      /* out.write(byte[] b,int off, int len)
      ** Writes len bytes from the specified byte array starting at offset off to this output stream.
      */
      out.write(buffer, 0, bytesRead);
    } 
       
    in.close();
    out.close();
  }
}

/****************************************************
 * 作者：孙卫琴                                     *
 * 来源：<<Tomcat与Java Web开发技术详解>>           *
 * 技术支持网址：www.javathinker.net                *
 ***************************************************/

```



### 5.2 读写Cookie

​	客户端首次访问服务器时，服务器先在客户端存放包含客户的相关信息的Cookie,以后客户端每次请求访问服务器时，都会在HTTP请求数据中包含Cookie(**准确的说Cookie会包含在HTTP请求的请求头中**),服务器解析HTTP请求中的Cookie,就能由此获得关于客户的信息

![image-20210611181857060](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210611181857060.png)

​	Tomcat作为Web服务器，提供了与Cookie交互的API，即 **java.servlet.http.Cookie**，因此Web应用中的Servlet可以直接通过Cookie类来访问HTTP请求中包含的Cookie中的数据。

```java
package mypack;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;

public class CookieServlet extends HttpServlet {
  int count=0;

  public void doGet(HttpServletRequest request,HttpServletResponse response)
    throws ServletException, IOException {
    
    response.setContentType("text/plain");
    PrintWriter out=response.getWriter();
    //获得HTTP请求中的所有Cookie
    Cookie[] cookies=request.getCookies();
    if(cookies!=null){
      //访问每个Cookie
      for(int i = 0; i < cookies.length; i++){
        out.println("Cookie name:"+cookies[i].getName());
        out.println("Cookie value:"+cookies[i].getValue());
        out.println("Max Age:"+cookies[i].getMaxAge()+"\r\n");
      }
    }else{
      out.println("No cookie.");
    } 
 
    //向客户端写一个Cookie
    //Cookie类的构造方法有两个参数，cookieName和cookieValue,都是字符串类型
    response.addCookie(new Cookie(
            "cookieName" + count, "cookieValue" + count));
    count++;
  }
}

/* Cookie类一些其他常用的方法
** cookie.setMaxAge(60*60)		//设置客户端在本地保存该Cookie的时间1h，想要删除cookie直接将参数改为0即可
** cookie.setValue(string value) //修改cookie的值，cookieName是final类型的，不能修改
*/

/* HTTPServletResponse提供的与Cookie相关的方法
** response.addCookie(Cookie cookie) //将该cookie加入到响应中
*/
/* HTTPServletRequest提供的与Cookie相关的方法
** Cookie[] response.getCookies()	//返回HTTP请求中的所有Cookie，将其作为一个数组返回
*/


/****************************************************
 * 作者：孙卫琴                                     *
 * 来源：<<Tomcat与Java Web开发技术详解>>           *
 * 技术支持网址：www.javathinker.net                *
 ***************************************************/

```

 **Cookie的共享范围**

 默认情况下，哪个web应用保存的cookie，cookie就只在这个web应用中可以访问。

**在同一个Tomcat容器中共享Cookie**

一个Servlet容器可以包含多个web应用，为了使同一个Servlet中的其他web应用也能访问该cookie

```java
Cookie cookie=new Cookie("username","tom");
cookie.setPath("/");
response.addCookie(cookie);
```

setPath（string path）方法:    Specifies a path for the cookie to which the client should send the cookie.

"/" 表示Tomcat服务器的根路径，相当于 **localhost:8080/** 或者 **webapp/** ,因此将cookie的path设置为 “/” 之后，客户端访问该浏览器中的其他web应用的时候也会发送该cookie



**在不同的Tomcat容器中共享Cookie**

```java
Cookie cookie=new Cookie("username","tom");
cookie.setDomain(".cat.tom");
response.addCookie(cookie);
```

setDomain（string pattern）方法：Specifies the domain within which this cookie should be presented.



### 5.3 访问Web应用的工作目录

​	每个Web应用都有一个工作目录，Servlet容器会把与这个Web应用相关的临时文件存放在这个目录下，Tomcat为Web应用提供的默认的工作目录为

```xml
<CATALINA_HOME>/work/[enginename]/[hostname]/[contextpath]
```

​	例如：web应用helloapp被默认发布到Tomcat的名为Catalina的Engine的localhost虚拟主机中，那么helloapp应用的默认工作目录为：

```xml
<CATALINA_HOME>/work/Catalina/localhost/helloapp
```

**设置work directory**

​	web应用的工作目录可以在server.xml文件中的context标签中设置，通过workDir属性来设置

**在Servlet中获得work directory**

​	当Servlet容器在初始化一个Web应用时，会向刚创建的ServletContext对象中设置一个名为 **“javax.servlet.context.tempdir” 的属性，**该属性值为一个**java.io.File类型**的对象，代表当前Web应用的工作目录，因此，Servlet获取工作目录的方式为：

```java
File workDir=(File)context.getAttribute(“javax.servlet.context.tempdir”);
```



### 5.4 转发和包含

​	**Servlet对象由Servlet容器创建，并且Servlet对象的service()方法也有容器调用，而一个Servlet无法直接调用另一个Servlet对象的service()方法，因为这个Servlet对象无法获得另一个Servlet对象的引用**

​	**请求转发**和**包含**使一个Servlet对象可以调用另一个Servlet对象的方法

​	

​	java.servlet.RequestDispatcher类：

​		该类表示请求分发器，包含**include()**和**forward()**两个方法，分别表示请求和转发

```java
void forward(ServletRequest request, ServletResponse response)
void include(ServletRequest request, ServletResponse response)
```

​	可以看到forward方法和include方法都是包含两个参数 ServletRequest和ServletResponse的， 因此被请求或者包含的Servlet与原Servlet共享 request和 response对象

​		**获取RequestDispatcher对象的方法：**

```java
//方式一：调用ServletContext的getRequestDispatcher(String path)方法
	//其中path表示被请求或者包含的Servlet的绝对路径，绝对路径以 “/” 打头，“/” 表示 localhost:8080/ 或者 webapp/
//方式二：调用ServletRequest的getRequestDispatcher(String path)方法
	//其中path表示被请求或者包含的Servlet的绝对路径或者相对路径,绝对路径同上，相对路径表示原Servlet所在的目录
```



 	**dispatcher.forward(request,response)**方法的处理流程：

​		a.清空存放响应正文数据的缓冲去  **因此原组件Servlet中已经通过PrintWriter或者ServletOutputStream写入响应正文中的内容会被清空（响应头中的内容是否会清空不知道，请试一试）。如果在调用forward之前调用了flushBuffer()或者close()方法，程序会抛出IllegalStateException异常**

​		b.如果目标组件为Servlet或者JSP，调用其service()方法，把该方法中的响应结果发送给客户端；如果是静态的HTML文档，就读取文档中的数据并把它发送到客户端

​		

​	**dispatcher.include(request,response)**方法的处理流程：

​		a.如果目标组件为Servlet或者JSP，调用其service()方法，把该方法中的**响应结果添加到响应正文中**；如果是静态的HTML文档，就读取文档中的数据并**把它添加到响应正文中**

​		b.返回到源组件的服务方法中，继续执行后续的代码块



### 5.5 重定向 sendRedirect(String location)

​	重定向机制是由HTTP协议提供的一种机制，存在于HttpServletResponse接口中，而**ServletResponse接口中不包含该方法**

​	**重定向的运作流程：**

​		1.用户在浏览器输入特定的URL，请求访问服务器端的某个组件

​		2.服务器端的组件返回一个状态码为302的响应结果，该响应结果的含义为：让浏览器端再请求访问另一个Web组件。响应结果中提供了另一个Web组件的URL，另一个Web组件可以在同一个Web服务器上，也可以在不同Web服务器上

​		3.浏览器接受到这种响应结果之后，再立即自动请求访问另一个Web组件

​		4.浏览器接受来自另一个Web组件的响应结果



​	**HttpServletResponse.sendRedirect(String location)方法：**

​		Servlet源组件生成的响应结果不会被发送到客户端，只有重定向的目标组件生成的响应结果才会被发送到客户端

​		如果在调用sendRedirect之前调用了flushBuffer()或者close()方法，程序会抛出IllegalStateException异常

​		源组件和目标组件不共享ServletRequest对象，因为浏览器重新发出请求

​		sendRedirect方法中的 **location参数**，如果**以 “/” 打头，表示当前服务器根路径的URL** ， 如果**以“http://” 开头，表示完整的URL**

​		目标组件不需要是同一个Web服务器上的Web应用，可以是任意服务器上的





### 5.6 访问Servlet容器内的其他Web应用

​	同一个Servlet容器中可以有多个Web应用，Web应用之间是可以互相访问的

​	ServletContext接口中提供了**getContext(String uripath)** 方法，**该方法可以获取其他Web应用的ServletContext对象，参数 uripath 指定其他Web应用的URL入口**

​	可以通过**&lt;Context>元素的crossContext属性**设置Web应用是否能访问其他Web应用资源

​			crossContext=false: Context元素对应的Web应用无法得到其他Web应用的ServletContext对象

​			crossContext=true: Context元素对应的Web应用可以得到其他Web应用的ServletContext对象



### 5.7 解决并发问题

​	一个Web应用可能在一个时间被多个浏览器请求服务，因此可能会产生并发的问题

​	解决并发的方案：

​	**1.合理决定在Servlet中定义的变量的作用域类型**：

```java
package mypack;

import javax.servlet.*;
import java.io.*;

public class HelloServlet1 extends GenericServlet {
  private String username=null;  //username为实例变量
  
  /** 响应客户请求*/
  public void service(ServletRequest request,
              ServletResponse response)
              throws ServletException, IOException {
    
    //把username请求参数赋值给username实例变量 
    username=request.getParameter("username");
    
    try{
      Thread.sleep(3000); //特意延长响应客户请求的时间
    }catch(Exception e){e.printStackTrace();}
      
    //设置HTTP响应的正文的MIME类型及字符编码
    response.setContentType("text/html;charset=GBK");
   
    /*输出HTML文档*/
    PrintWriter out = response.getWriter();
    out.println("<html><head><title>HelloServlet</TITLE></head>");
    out.println("<body>");
    out.println("你好: "+username);
    out.println("</body></html>");
     
    out.close(); //关闭PrintWriter
   }
}


```

​		在以上程序中，通过两个浏览器同时访问该Web应用

​		浏览器1： http://localhost:8080/helloapp/hello1?username=老鼠

​		浏览器2： http://localhost:8080/helloapp/hello1?username=小鸭

​		可能两个浏览器得到的结果都是：

​				**你好：小鸭**

​		这是由于并发问题导致的。



​		解决方案：将username的定义放到service方法中，这样不同浏览器请求web服务的时候会有不同的username变量



​	**2.使用Java同步机制对多线程同步**

