首先用maven创建一个web项目，创建好的项目结构如下

![image-20210915163109144](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210915163109144.png)

如果缺少java和resources文件夹，可以自动创建。



接下来看看完整版的SSM的结构目录

![](C:\Users\haotian wei\AppData\Roaming\Typora\typora-user-images\image-20210915163003892.png)

java文件夹：存放源码

+ controller: 这个文件夹放SpringMVC的控制器

+ dao/mapper: 这个文件夹放mybatis的映射器，一个映射器由一个接口和一个XML文件组成

+ pojo: 存放类，这里的类跟数据库里面的表一一对应

+ service: 存放业务类，service中的业务类会用到dao中的映射器来与数据库交互，并且这个业务类会被controller中的类调用

resources文件夹：存放各种配置文件

+ application-context.xml: Spring的配置文件，在这里一般会配置mybatis中SqlSessionFactory的bean、事务等等等

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
  
      <!--spring的配置文件：声明service ，dao， 工具类， 事务配置-->
  
      <context:component-scan base-package="edu.zju.wht" />
  
      <context:property-placeholder location="classpath:/jdbc.properties" />
  
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
            init-method="init" destroy-method="close">
          <property name="url" value="${jdbc.url}" />
          <property name="username" value="${jdbc.username}" />
          <property name="password" value="${jdbc.password}" />
      </bean>
  
      <bean id="factory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource"/>
          <property name="configLocation" value="classpath:/mybatis.xml" />
      </bean>
  
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <property name="sqlSessionFactoryBeanName" value="factory" />
          <property name="basePackage" value="edu.zju.wht.dao" />
      </bean>
  
      <!--事务配置-->
  </beans>
  ```

  

+ dispatcherServlet.xml:springmvc的配置文件： 声明controller， 视图解析器等web开发中的的对象

  ```:xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
  
      <!--springmvc的配置文件： 声明controller， 视图解析器等web开发中的的对象-->
  
      <context:component-scan base-package="edu.zju.wht.controller" />
  
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="prefix" value="/WEB-INF/jsp/" />
          <property name="suffix" value=".jsp" />
  
      </bean>
  
      <!--
        作用：
         1. 创建HttpMessageConverter接口的7个实现类对象， 处理java对象到json的转换
         2. 解决静态资源中，动态资源访问失败的问题。
      -->
      <mvc:annotation-driven />
  </beans>
  ```

+ jdbc.properties: 数据库的配置信息

```xml
jdbc.url=jdbc:mysql://localhost:3306/springdb?useUnicode=true&characterEncoding=utf-8
jdbc.username=root
jdbc.password=
```

+ mybatis.xml: mybatis的配置信息

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <!--设置日志-->
      <settings>
          <setting name="logImpl" value="STDOUT_LOGGING"/>
      </settings>
  
  
      <typeAliases>
          <!--domain包中的类名就是别名-->
          <package name= "edu.zju.wht"/>
      </typeAliases>
  
  
      <mappers>
          <!--加载dao包中的所有mapper文件-->
          <package name="edu.zju.wht" />
      </mappers>
  </configuration>
  ```

webapp文件： 存放html、jsp、css、javascript等视图相关的文件

+ web.xml文件：web容器相关的配置文件，在这里配置dispatcherServlet

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

  <display-name>Archetype Created Web Application</display-name>
  <!--声明spring监听器-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:/application-context.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>


  <!--声明springmvc的中央调度器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:/dispatcherServlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
</web-app>

```

