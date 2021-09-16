# 1.SpringMVC流程和初始化

## 1.1 SpringMVC流程

![springMVC请求流程详解- 梦醒点灯- 博客园](https://images2015.cnblogs.com/blog/791227/201611/791227-20161125140338768-995727439.png)

SpringMVC启动的时候会去解析@Controller注解，然后生成对应的URI和请求的映射关系，并注册对应的方法。请求到来的时候，首先根据URI找到对应的HandlerMapping，然后组织为一个执行链，通过请求类型找到HandlerAdapter,通过这个HandlerAdapter执行HandlerExecutionChain的内容，最终在Controller的方法中将视图和模型（Model，即数据）返回给DispatcherServlet. DispatcherServlet会把对应的试图信息传递给视图解析器（视图解析器就是给view加了前缀和后缀，这样就能找到view的实际位置），得到视图的实际位置，然后将模型渲染到视图中去，最后响应用户请求。



## 1.2 SpringMVC的初始化

首先明白一件事情，DispatcherServlet是一个Servlet，它间接继承于HttpServlet。所以DispatcherServlet有所属的Web容器（不知道为什么可以参考JAVA EE中的Servlet技术上篇）。

那么DispathcerServlet所属的Web容器(称为Spring IoC容器)何时初始化呢？

DispatcherServlet初始化的时候会对Spring IoC容器进行初始化；通过ContextLoaderListener监听器也可以实现对Spring IoC容器的初始化。由于不止是DispatcherServlet需要用到Spring IoC的资源，其他组件也可能需要，并且不应该在用户请求的时候(即DispathcertServlet初始化的时候，不设置它的初始化时机那么第一次用户请求的时候它才会初始化)让用户等那么久，所以应该用ContextLoaderListener来进行初始化。

**使用ContextLoaderListener初始化Spring IoC容器**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.1" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd">
    <!-- 配置Spring IoC配置文件路径 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <!-- 配置ContextLoaderListener用以初始化Spring IoC容器 -->
    <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 配置DispatcherServlet -->
    <servlet>
        <!-- 注意：Spring MVC框架会根据servlet-name配置，找到/WEB-INF/dispatcher-servlet.xml作为配置文件载入Web工程中 -->
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 使得Dispatcher在服务器启动的时候就初始化 -->
        <load-on-startup>2</load-on-startup>
    </servlet>
    <!-- Servlet拦截配置 -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

初始化了Spring IoC容器之后，再初始化DispatcherServlet.在DispatcherServlet初始化期间，可能或者一定会（没读懂源码，这块不确定）初始化SpringMVC的各个组件，包括上面那张图上的各个部件以及一些其他部件。

正因为DispatcherServlet会加载初始化这些部件，所以我们不需要很多配置就能够使用SpringMVC.



## 1.3 Spring MVC开发流程

### 1.3.1 控制器的开发

控制器Controller的开发是Spring MVC的核心内容，包括以下三个步骤

+ 获取请求参数
+ 处理业务逻辑
+ 绑定模型和视图

#### 1.3.1.1 获取请求参数

获取请求参数的方法有很多，但是下面这种不建议使用

```java
//表明URI是/index的时候该方法才请求
@RequestMapping(value="/index2",method=RequestMethod.GET)
public ModelAndView  index2(HttpSession session,HttpServletRequest request) {
    //模型和视图
    ModelAndView mv = new ModelAndView();  
    //视图逻辑名称为index
    mv.setViewName("index");
    //返回模型和视图
    return mv;
}
```

SpringMVC会解析代码中的参数session和request, 然后将传递servlet容器的这两个对象。这种方法可以很容易的获取Http请求种的信息，但是坏处就是与Servlet容器紧密关联，不利于扩展和测试。

Spring MVC提供了诸多的注解来解析参数，其目的在于把控制器从复杂的Servlet API中玻璃，这样就可以在非Servlet容器环境中重用控制器，也方便测试人员对其进行有效测试。

+ 接受普通请求参数

  + 逐个接受请求参数

  ```java
   /**
       * 逐个接收请求参数
       * 要求： 请求中的参数名和控制器方法的形参名一样。 按名称对象接收请求参数
       *
       * 参数接收：
       *  1. 框架使用request对象，接收参数
       *     String name = request.getParameter("name");
       *     String age = request.getParameter("age");
       *  2. 在中央调度器的内部调用 doProperParam方法时， 按名称对象传递参数
       *      doPropertyParam(name, Integer.valueOf(age) )
       *     框架可以实现请求参数 String 到int ， long ，float ，double等类型的转换。
       */
      @RequestMapping(value ="/receive-property.do")
      public ModelAndView doPropertyParam(String name, Integer age) {
  
          System.out.println("执行了MyController的doPropertyParam方法name=" + name+",age="+age);
          ModelAndView mv  = new ModelAndView();
          //添加数据
          mv.addObject("myname", name);
          mv.addObject("myage", age);
  
          mv.setViewName("show");
  
          //返回结果
          return mv;
  
      }
  ```

  + 使用对象接受请求参数

  ```java
      /**
       * 使用对象接收请求中的参数
       * 要求： 参数名和java对象的属性名一样。
       *       java类需要有一个无参数构造方法， 属性有set方法
       *
       * 框架的处理：
       *  1. 调用Student的无参数构造方法，创建对象
       *  2. 调用对象set方法， 同名的参数，调用对应的set方法。
       *     参数是name ，调用setName(参数值)
       */
      @RequestMapping("/receive-object.do")
      public ModelAndView doReceiveObject(Student student){
          System.out.println("MyController的方法doReceiveObject="+student);
          ModelAndView mv = new ModelAndView();
          mv.addObject("myname", student.getName());
          mv.addObject("myage", student.getAge());
          mv.setViewName("show");
          return mv;
      }
  ```

  相比于逐个接受请求参数而言，使用对象来接受参数适用于参数比较多的情况

  注意接受普通参数请求运行请求参数为空



+ 使用@RequestParam注解获取参数

```java
 /**
     * 逐个接收请求参数， 请求中参数名和形参名不一样
     * @RequestParam : 解决名称不一样的问题
     *       属性： value 请求中的参数名称
     *              required : boolean类型的，默认是true
     *                    true：请求中必须有此参数，没有报错。
     *                    false：请求中可以没有此参数。
     *       位置： 在形参定义的前面
     */
    @RequestMapping(value ="/receive-param.do")
    public ModelAndView doReceiveParam(@RequestParam(value = "rname",required = false) String name,
                                       @RequestParam(value = "rage",required = false) Integer age) {

        System.out.println("执行了MyController的doReceiveParam方法name=" + name+",age="+age);
        ModelAndView mv  = new ModelAndView();
        //添加数据
        mv.addObject("myname", name);
        mv.addObject("myage", age);

        mv.setViewName("show");

        //返回结果
        return mv;

    }
```

@RequestParam用于解决请求中参数名称和形参名不一致的情况



+ 使用URL传递传输——需要通过@RequestMapping和@PathVariable两个注解共同完成

```java
//注入角色服务对象
	@Autowired
	RoleService roleService;

	//{id}代表接收一个参数
	@RequestMapping("/getRole/{id}")
	//注解@PathVariable表示从URL的请求地址中获取参数
	public ModelAndView pathVariable(@PathVariable("id") Long id)  {
		Role role = roleService.getRole(id);
		ModelAndView mv = new ModelAndView();
		//绑定数据模型
		mv.addObject(role);
		//设置为JSON视图
		mv.setView(new MappingJackson2JsonView());
		return mv;
	}
```

对于上面所示例子，我们期望URL写作 /getRole/1 , 其中1就是参数，对应{id}.

{id}代表处理器需要接受一个组成URL的参数，且参数名称为id，那么方法中的@PathVariable("id")表示将获取这个在@RequestMapping中定义的名称为id的参数，这样就可以在方法内获取这个参数了。

**注意@PathVariable允许对应的参数为空**



### 1.3.2 @RequestMapping

@RequestMapping注解源码

```JAVA
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}

```

