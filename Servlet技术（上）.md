## 1. Servlet API

Servlet API 文档地址

[请点击我]: https://tomcat.apache.org/tomcat-9.0-doc/servletapi/index.html

Servlet API的源文件在apache-tomcat安装路径下的/lib/servlet-api.jar中



Servlet API主要由两个Java包组成：**javax.servlet和javax.servlet.http**

其中javax.servlet包中定义了Servlet接口

![20200327083146203](D:\Users\haotian wei\Desktop\20200327083146203.png)



servlet接口

GenericServlet抽象类

HttpServlet抽象类

ServletRequest接口

HttpServletRequest接口

ServletResponse接口

HttpServletResponse接口

ServletConfig接口

ServletContext接口



**Servlet接口**

​	供Servlet容器调用的方法:

​		init(ServletConfig config) ：初始化Servlet对象，容器创建好Servlet对象后，就调用该方法

​		service(ServletRequest req,ServletResponse res) ：负责响应客户的请求，为客户提供相应的服务。容器接收到客户端要访问的特定Servlet对象的请求时，就会调用改方法

​		destory() ：负责释放Servlet对象占用的资源，当Servlet对象结束生命周期时，容器会调用此方法

​	供Java Web应用中的程序代码访问的方法：

​		getServletConfig() : 返回一个ServletConfig对象，改对象包含了Servlet的初始化参数信息

​		getServletInfo() : 返回一个字符串，该字符串中包含了Servlet的创建者、版本和版权等信息

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.io.IOException;

public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}

```



**GenericServlet抽象类**	（运用了装饰设计模式）

​	该类提供了Servlet接口的通用实现，还实现了ServletConfig接口和Serializable接口，并且该抽象类**有一个ServletConfig类型的私有实例变量config**,从而使得GenericServlet抽象类与ServletConfig对象关联

​	GenericServlet类没有实现service（）方法，service（）方法是该类中唯一的抽象方法，GenericServlet类的子类必须实现该方法。

​	GenericSeervlet类实现类ServletConfig接口中的所有方法，因此GenericServlet及其子类可以直接调用在ServletConfig接口中定义的getServletContext()、getInitParameter()、getInitParameterNames()等方法

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.io.IOException;
import java.io.Serializable;
import java.util.Enumeration;

public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final long serialVersionUID = 1L;
    private transient ServletConfig config;

    public GenericServlet() {
    }

    public void destroy() {
    }

    public String getInitParameter(String name) {
        return this.getServletConfig().getInitParameter(name);
    }

    public Enumeration<String> getInitParameterNames() {
        return this.getServletConfig().getInitParameterNames();
    }

    public ServletConfig getServletConfig() {
        return this.config;
    }

    public ServletContext getServletContext() {
        return this.getServletConfig().getServletContext();
    }

    public String getServletInfo() {
        return "";
    }

    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }

    public void init() throws ServletException {
    }

    public void log(String message) {
        this.getServletContext().log(this.getServletName() + ": " + message);
    }

    public void log(String message, Throwable t) {
        this.getServletContext().log(this.getServletName() + ": " + message, t);
    }

    public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    public String getServletName() {
        return this.config.getServletName();
    }
}

```



**HttpServlet抽象类**

​	该类是GenericServlet抽象类的子类，该类为Servlet接口提供了与HTTP协议相关的通用实现，因此在开发基于HTTP协议的Java Web应用的时候，**自定义的Servlet类一般都继承该类而不是GenericServlet类**

​	HttpServlet类实现类service()方法，在该service(ServletRequest req,ServletResponse resp)方法中，将参数ServletRequest和ServletResponse转换成HttpServletRequset和HttpRequestResponse, 然后调用重载的service(HttpServletRequest req,HttpServletResponse resp)方法。

​	在service(HttpServletRequest req,HttpServletResponse resp)方法中，根据req获取客户端的请求方法（GET、POST、PUT、DELETE等），对于不同的请求方法，调用相应的doGet、doPost等方法，req和resp两个参数也会传给doGet或者doPost方法。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet.http;

import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.text.MessageFormat;
import java.util.Enumeration;
import java.util.ResourceBundle;
import javax.servlet.DispatcherType;
import javax.servlet.GenericServlet;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public abstract class HttpServlet extends GenericServlet {
    private static final long serialVersionUID = 1L;
    private static final String METHOD_DELETE = "DELETE";
    private static final String METHOD_HEAD = "HEAD";
    private static final String METHOD_GET = "GET";
    private static final String METHOD_OPTIONS = "OPTIONS";
    private static final String METHOD_POST = "POST";
    private static final String METHOD_PUT = "PUT";
    private static final String METHOD_TRACE = "TRACE";
    private static final String HEADER_IFMODSINCE = "If-Modified-Since";
    private static final String HEADER_LASTMOD = "Last-Modified";
    private static final String LSTRING_FILE = "javax.servlet.http.LocalStrings";
    private static final ResourceBundle lStrings = ResourceBundle.getBundle("javax.servlet.http.LocalStrings");

    public HttpServlet() {
    }

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_get_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected long getLastModified(HttpServletRequest req) {
        return -1L;
    }

    protected void doHead(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        if (DispatcherType.INCLUDE.equals(req.getDispatcherType())) {
            this.doGet(req, resp);
        } else {
            NoBodyResponse response = new NoBodyResponse(resp);
            this.doGet(req, response);
            response.setContentLength();
        }

    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_post_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected void doPut(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_put_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    protected void doDelete(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String protocol = req.getProtocol();
        String msg = lStrings.getString("http.method_delete_not_supported");
        if (protocol.endsWith("1.1")) {
            resp.sendError(405, msg);
        } else {
            resp.sendError(400, msg);
        }

    }

    private static Method[] getAllDeclaredMethods(Class<?> c) {
        if (c.equals(HttpServlet.class)) {
            return null;
        } else {
            Method[] parentMethods = getAllDeclaredMethods(c.getSuperclass());
            Method[] thisMethods = c.getDeclaredMethods();
            if (parentMethods != null && parentMethods.length > 0) {
                Method[] allMethods = new Method[parentMethods.length + thisMethods.length];
                System.arraycopy(parentMethods, 0, allMethods, 0, parentMethods.length);
                System.arraycopy(thisMethods, 0, allMethods, parentMethods.length, thisMethods.length);
                thisMethods = allMethods;
            }

            return thisMethods;
        }
    }

    protected void doOptions(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Method[] methods = getAllDeclaredMethods(this.getClass());
        boolean ALLOW_GET = false;
        boolean ALLOW_HEAD = false;
        boolean ALLOW_POST = false;
        boolean ALLOW_PUT = false;
        boolean ALLOW_DELETE = false;
        boolean ALLOW_TRACE = true;
        boolean ALLOW_OPTIONS = true;
        Class clazz = null;

        try {
            clazz = Class.forName("org.apache.catalina.connector.RequestFacade");
            Method getAllowTrace = clazz.getMethod("getAllowTrace", (Class[])null);
            ALLOW_TRACE = (Boolean)getAllowTrace.invoke(req, (Object[])null);
        } catch (NoSuchMethodException | SecurityException | IllegalAccessException | IllegalArgumentException | InvocationTargetException | ClassNotFoundException var14) {
        }

        for(int i = 0; i < methods.length; ++i) {
            Method m = methods[i];
            if (m.getName().equals("doGet")) {
                ALLOW_GET = true;
                ALLOW_HEAD = true;
            }

            if (m.getName().equals("doPost")) {
                ALLOW_POST = true;
            }

            if (m.getName().equals("doPut")) {
                ALLOW_PUT = true;
            }

            if (m.getName().equals("doDelete")) {
                ALLOW_DELETE = true;
            }
        }

        String allow = null;
        if (ALLOW_GET) {
            allow = "GET";
        }

        if (ALLOW_HEAD) {
            if (allow == null) {
                allow = "HEAD";
            } else {
                allow = allow + ", HEAD";
            }
        }

        if (ALLOW_POST) {
            if (allow == null) {
                allow = "POST";
            } else {
                allow = allow + ", POST";
            }
        }

        if (ALLOW_PUT) {
            if (allow == null) {
                allow = "PUT";
            } else {
                allow = allow + ", PUT";
            }
        }

        if (ALLOW_DELETE) {
            if (allow == null) {
                allow = "DELETE";
            } else {
                allow = allow + ", DELETE";
            }
        }

        if (ALLOW_TRACE) {
            if (allow == null) {
                allow = "TRACE";
            } else {
                allow = allow + ", TRACE";
            }
        }

        if (ALLOW_OPTIONS) {
            if (allow == null) {
                allow = "OPTIONS";
            } else {
                allow = allow + ", OPTIONS";
            }
        }

        resp.setHeader("Allow", allow);
    }

    protected void doTrace(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String CRLF = "\r\n";
        StringBuilder buffer = (new StringBuilder("TRACE ")).append(req.getRequestURI()).append(" ").append(req.getProtocol());
        Enumeration reqHeaderEnum = req.getHeaderNames();

        while(reqHeaderEnum.hasMoreElements()) {
            String headerName = (String)reqHeaderEnum.nextElement();
            buffer.append(CRLF).append(headerName).append(": ").append(req.getHeader(headerName));
        }

        buffer.append(CRLF);
        int responseLength = buffer.length();
        resp.setContentType("message/http");
        resp.setContentLength(responseLength);
        ServletOutputStream out = resp.getOutputStream();
        out.print(buffer.toString());
        out.close();
    }

    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }

    private void maybeSetLastModified(HttpServletResponse resp, long lastModified) {
        if (!resp.containsHeader("Last-Modified")) {
            if (lastModified >= 0L) {
                resp.setDateHeader("Last-Modified", lastModified);
            }

        }
    }

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }

        this.service(request, response);
    }
}

```



**ServletRequest接口**

​	Servlet接口的service()方法有两个参数，其中一个就是该接口类型的参数，ServletRequest接口的具体实现类由Servlet容器传入，即service服务被调用的时候Tomcat容器会将一个ServletRequest的实现类作为参数传给service方法

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.Enumeration;
import java.util.Locale;
import java.util.Map;

public interface ServletRequest {
    Object getAttribute(String var1);

    Enumeration<String> getAttributeNames();

    String getCharacterEncoding();

    void setCharacterEncoding(String var1) throws UnsupportedEncodingException;
	//返回请求正文的长度，如果长度未知，返回-1
    int getContentLength();

    long getContentLengthLong();
	//返回请求正文的MIME类型，如果未知，返回null
    String getContentType();
	//返回用于读取请求正文的输入流
    ServletInputStream getInputStream() throws IOException;
	//根据参数名var1，返回来自客户请求中匹配的请求参数值
    String getParameter(String var1);

    Enumeration<String> getParameterNames();

    String[] getParameterValues(String var1);

    Map<String, String[]> getParameterMap();

    String getProtocol();

    String getScheme();

    String getServerName();

    int getServerPort();

    BufferedReader getReader() throws IOException;

    String getRemoteAddr();

    String getRemoteHost();

    void setAttribute(String var1, Object var2);

    void removeAttribute(String var1);

    Locale getLocale();

    Enumeration<Locale> getLocales();

    boolean isSecure();

    RequestDispatcher getRequestDispatcher(String var1);

    /** @deprecated */
    @Deprecated
    String getRealPath(String var1);

    int getRemotePort();
	//返回服务器的主机名
    String getLocalName();

    String getLocalAddr();

    int getLocalPort();

    ServletContext getServletContext();

    AsyncContext startAsync() throws IllegalStateException;

    AsyncContext startAsync(ServletRequest var1, ServletResponse var2) throws IllegalStateException;

    boolean isAsyncStarted();

    boolean isAsyncSupported();

    AsyncContext getAsyncContext();

    DispatcherType getDispatcherType();
}

```



**HttpServletRequest接口**

​	该接口是Servlet接口的子接口，HttpServlet类的重载service()方法以及doGet()和doPost()方法都有一个该类型的参数

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet.http;

import java.io.IOException;
import java.security.Principal;
import java.util.Collection;
import java.util.Collections;
import java.util.Enumeration;
import java.util.Map;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;

public interface HttpServletRequest extends ServletRequest {
    String BASIC_AUTH = "BASIC";
    String FORM_AUTH = "FORM";
    String CLIENT_CERT_AUTH = "CLIENT_CERT";
    String DIGEST_AUTH = "DIGEST";

    String getAuthType();

    Cookie[] getCookies();

    long getDateHeader(String var1);

    String getHeader(String var1);

    Enumeration<String> getHeaders(String var1);

    Enumeration<String> getHeaderNames();

    int getIntHeader(String var1);

    default HttpServletMapping getHttpServletMapping() {
        return new HttpServletMapping() {
            public String getMatchValue() {
                return "";
            }

            public String getPattern() {
                return "";
            }

            public String getServletName() {
                return "";
            }

            public MappingMatch getMappingMatch() {
                return null;
            }
        };
    }

    String getMethod();

    String getPathInfo();

    String getPathTranslated();

    default PushBuilder newPushBuilder() {
        return null;
    }

    String getContextPath();

    String getQueryString();

    String getRemoteUser();

    boolean isUserInRole(String var1);

    Principal getUserPrincipal();

    String getRequestedSessionId();

    String getRequestURI();

    StringBuffer getRequestURL();

    String getServletPath();

    HttpSession getSession(boolean var1);

    HttpSession getSession();

    String changeSessionId();

    boolean isRequestedSessionIdValid();

    boolean isRequestedSessionIdFromCookie();

    boolean isRequestedSessionIdFromURL();

    /** @deprecated */
    @Deprecated
    boolean isRequestedSessionIdFromUrl();

    boolean authenticate(HttpServletResponse var1) throws IOException, ServletException;

    void login(String var1, String var2) throws ServletException;

    void logout() throws ServletException;

    Collection<Part> getParts() throws IOException, ServletException;

    Part getPart(String var1) throws IOException, ServletException;

    <T extends HttpUpgradeHandler> T upgrade(Class<T> var1) throws IOException, ServletException;

    default Map<String, String> getTrailerFields() {
        return Collections.emptyMap();
    }

    default boolean isTrailerFieldsReady() {
        return false;
    }
}

```



**ServletResponse接口**

​	Servlet接口的service()方法中有一个该类型的参数，Servlet通过ServletResponse对象类生成响应结果，该接口的实例对象由Servlet容器创建。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Locale;

public interface ServletResponse {
    String getCharacterEncoding();

    String getContentType();
	//返回一个ServletOutputStream对象，用它来将二进制数据写入缓冲区，缓冲区的数据最后会被返回给客户端，记得要用flush()方法或者close()方法
    ServletOutputStream getOutputStream() throws IOException;
	//返回一个PrintWriter对象，用它来将字符串形式的数据写入缓冲区，缓冲区的数据最后会被返回给客户端，记得要用flush()方法或者close()方法
    PrintWriter getWriter() throws IOException;
	//设置响应正文的编码
    void setCharacterEncoding(String var1);
	//设置响应正文长度
    void setContentLength(int var1);

    void setContentLengthLong(long var1);
	//设置响应正文MIME类型
    void setContentType(String var1);

    void setBufferSize(int var1);

    int getBufferSize();

    void flushBuffer() throws IOException;

    void resetBuffer();

    boolean isCommitted();

    void reset();

    void setLocale(Locale var1);

    Locale getLocale();
}

```



**注意：设置响应正文的编码、长度或者MIME类型之后，需要调用PrintWriter或者ServletOutputStream的flush方法或者close方法设置才会生效**



**HttpServletResponse接口**

​	该接口是ServletResonse接口的子接口，HttpServlet的重载service()方法以及doGet、doPost()方法都有一个该类型的参数

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet.http;

import java.io.IOException;
import java.util.Collection;
import java.util.Map;
import java.util.function.Supplier;
import javax.servlet.ServletResponse;

public interface HttpServletResponse extends ServletResponse {
    int SC_CONTINUE = 100;
    int SC_SWITCHING_PROTOCOLS = 101;
    int SC_OK = 200;
    int SC_CREATED = 201;
    int SC_ACCEPTED = 202;
    int SC_NON_AUTHORITATIVE_INFORMATION = 203;
    int SC_NO_CONTENT = 204;
    int SC_RESET_CONTENT = 205;
    int SC_PARTIAL_CONTENT = 206;
    int SC_MULTIPLE_CHOICES = 300;
    int SC_MOVED_PERMANENTLY = 301;
    int SC_MOVED_TEMPORARILY = 302;
    int SC_FOUND = 302;
    int SC_SEE_OTHER = 303;
    int SC_NOT_MODIFIED = 304;
    int SC_USE_PROXY = 305;
    int SC_TEMPORARY_REDIRECT = 307;
    int SC_BAD_REQUEST = 400;
    int SC_UNAUTHORIZED = 401;
    int SC_PAYMENT_REQUIRED = 402;
    int SC_FORBIDDEN = 403;
    int SC_NOT_FOUND = 404;
    int SC_METHOD_NOT_ALLOWED = 405;
    int SC_NOT_ACCEPTABLE = 406;
    int SC_PROXY_AUTHENTICATION_REQUIRED = 407;
    int SC_REQUEST_TIMEOUT = 408;
    int SC_CONFLICT = 409;
    int SC_GONE = 410;
    int SC_LENGTH_REQUIRED = 411;
    int SC_PRECONDITION_FAILED = 412;
    int SC_REQUEST_ENTITY_TOO_LARGE = 413;
    int SC_REQUEST_URI_TOO_LONG = 414;
    int SC_UNSUPPORTED_MEDIA_TYPE = 415;
    int SC_REQUESTED_RANGE_NOT_SATISFIABLE = 416;
    int SC_EXPECTATION_FAILED = 417;
    int SC_INTERNAL_SERVER_ERROR = 500;
    int SC_NOT_IMPLEMENTED = 501;
    int SC_BAD_GATEWAY = 502;
    int SC_SERVICE_UNAVAILABLE = 503;
    int SC_GATEWAY_TIMEOUT = 504;
    int SC_HTTP_VERSION_NOT_SUPPORTED = 505;

    void addCookie(Cookie var1);

    boolean containsHeader(String var1);

    String encodeURL(String var1);

    String encodeRedirectURL(String var1);

    /** @deprecated */
    @Deprecated
    String encodeUrl(String var1);

    /** @deprecated */
    @Deprecated
    String encodeRedirectUrl(String var1);

    void sendError(int var1, String var2) throws IOException;

    void sendError(int var1) throws IOException;

    void sendRedirect(String var1) throws IOException;

    void setDateHeader(String var1, long var2);

    void addDateHeader(String var1, long var2);

    void setHeader(String var1, String var2);

    void addHeader(String var1, String var2);

    void setIntHeader(String var1, int var2);

    void addIntHeader(String var1, int var2);

    void setStatus(int var1);

    /** @deprecated */
    @Deprecated
    void setStatus(int var1, String var2);

    int getStatus();

    String getHeader(String var1);

    Collection<String> getHeaders(String var1);

    Collection<String> getHeaderNames();

    default void setTrailerFields(Supplier<Map<String, String>> supplier) {
    }

    default Supplier<Map<String, String>> getTrailerFields() {
        return null;
    }
}

```



**ServletConfig接口**

​	Servlet接口的init方法中有一个ServletConfig类型的参数，GenericServlet实现Servlet中init方法的时候，将ServletConfig参数赋值给GenericServlet的ServletConfig类型的私有属性config。

​	ServletConfig对象中包含了Servlet的初始化信息，此为ServletConfig还与ServletContext对象相关联。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.util.Enumeration;

public interface ServletConfig {
    String getServletName();
	//返回一个ServletContext对象，该对象记录整个Web应用程序的相关信息
    ServletContext getServletContext();
	//获取xml配置文件中改Servlet的初始化信息,即init-param中的param-value中的值，而参数String var1对应param-name的值
    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();
}

```



```xml
<servlet>
    <servlet-name>Font</servlet-name>
    <servlet-class>mypack.FontServlet</servlet-class>
    <init-param>
        <param-name>color</param-name>
        <param-value>blue</param-value>
    </init-param>
        <init-param>
        <param-name>size</param-name>
        <param-value>15</param-value>
    </init-param>
</servlet>

<servlet-mapping>
    <servlet-name>Font</servlet-name>
    <url-pattern>/font</url-pattern>
</servlet-mapping>
```



**ServletContext接口**

​	该接口是Servlet与Servlet容器之间进行通信的接口，Servlet容器在启动一个Web应用时，会为它创建一个ServletContext对象，每个Web应用都有唯一的ServletContext对象。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package javax.servlet;

import java.io.InputStream;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.Enumeration;
import java.util.EventListener;
import java.util.Map;
import java.util.Set;
import javax.servlet.ServletRegistration.Dynamic;
import javax.servlet.descriptor.JspConfigDescriptor;

public interface ServletContext {
    String TEMPDIR = "javax.servlet.context.tempdir";
    String ORDERED_LIBS = "javax.servlet.context.orderedLibs";

    String getContextPath();

    ServletContext getContext(String var1);

    int getMajorVersion();

    int getMinorVersion();

    int getEffectiveMajorVersion();

    int getEffectiveMinorVersion();

    String getMimeType(String var1);

    Set<String> getResourcePaths(String var1);

    URL getResource(String var1) throws MalformedURLException;

    InputStream getResourceAsStream(String var1);

    RequestDispatcher getRequestDispatcher(String var1);

    RequestDispatcher getNamedDispatcher(String var1);

    /** @deprecated */
    @Deprecated
    Servlet getServlet(String var1) throws ServletException;

    /** @deprecated */
    @Deprecated
    Enumeration<Servlet> getServlets();

    /** @deprecated */
    @Deprecated
    Enumeration<String> getServletNames();

    void log(String var1);

    /** @deprecated */
    @Deprecated
    void log(Exception var1, String var2);

    void log(String var1, Throwable var2);

    String getRealPath(String var1);

    String getServerInfo();
	//获取web.xml配置文件中WebContext的初始值，即<context-param>标签中<param-value>中的值，参数var1对应<param-name>中的值
    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();

    boolean setInitParameter(String var1, String var2);
	//获取在Web应用范围内存储的数据的值
    Object getAttribute(String var1);

    Enumeration<String> getAttributeNames();
	//在Web应用范围内存入值
    void setAttribute(String var1, Object var2);

    void removeAttribute(String var1);

    String getServletContextName();

    Dynamic addServlet(String var1, String var2);

    Dynamic addServlet(String var1, Servlet var2);

    Dynamic addServlet(String var1, Class<? extends Servlet> var2);

    Dynamic addJspFile(String var1, String var2);

    <T extends Servlet> T createServlet(Class<T> var1) throws ServletException;

    ServletRegistration getServletRegistration(String var1);

    Map<String, ? extends ServletRegistration> getServletRegistrations();

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, String var2);

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Filter var2);

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Class<? extends Filter> var2);

    <T extends Filter> T createFilter(Class<T> var1) throws ServletException;

    FilterRegistration getFilterRegistration(String var1);

    Map<String, ? extends FilterRegistration> getFilterRegistrations();

    SessionCookieConfig getSessionCookieConfig();

    void setSessionTrackingModes(Set<SessionTrackingMode> var1);

    Set<SessionTrackingMode> getDefaultSessionTrackingModes();

    Set<SessionTrackingMode> getEffectiveSessionTrackingModes();

    void addListener(String var1);

    <T extends EventListener> void addListener(T var1);

    void addListener(Class<? extends EventListener> var1);

    <T extends EventListener> T createListener(Class<T> var1) throws ServletException;

    JspConfigDescriptor getJspConfigDescriptor();

    ClassLoader getClassLoader();

    void declareRoles(String... var1);

    String getVirtualServerName();

    int getSessionTimeout();

    void setSessionTimeout(int var1);

    String getRequestCharacterEncoding();

    void setRequestCharacterEncoding(String var1);

    String getResponseCharacterEncoding();

    void setResponseCharacterEncoding(String var1);
}

```





## 2. Java Web应用的声明周期

