# 1.Spring IoC的概念

## 1.1 Spring IoC概述

​	控制反转IoC是一种通过描述（在Java中可以是XML或者注解）并通过第三方（**Spring提供的IoC容器**）去产生或获取特定对象的方式。

​	在Spring中实现IoC（控制反转）的是IoC容器，其实现方式是依赖注入（Dependency Injection, DI）.

​	**控制反转最大的好处在于降低对象之间的耦合，当需要使用一个对象的时候，关于如何创造这个对象我们并不关心（创造对象由IoC容器来实现），我们只需要给出这个对象的描述信息，即可通过IoC容器获取这个对象。**

​	

## 1.2 Spring IoC容器

### 1.2.1 Spring IoC容器的设计

​	Spring IoC容器的设计主要是基于 **BeanFactory**和 **ApplicationContext**两个接口，其中ApplicationContext是BeanFactory的子接口之一， 其中BeanFactory是Spirng IoC容器所定义的最顶层的接口，而ApplicationContext是其子接口之一，并且对BeanFactory功能做了许多有用的扩展，所以**在绝大部分工作场景下，都会使用ApplicationContext作为Spring IoC容器。**

**ClassPathXmlApplicationContext是ApplicationContext接口的实现类，我们可以用它去实例化ApplicationContext接口**

示例：

**Bean的描述信息(XML方式，存储在applicationContext.xml文件中，这个文件是Spring的配置文件)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--声明bean-->
    <!--
       DI:给属性赋值
       简单类型: java中的基本数据类型和String

       1. set注入： spring调用类的set方法，通过set方法完成属性赋值
          简单类型的set注入：
          语法： <bean id="xxx" class="yyy">
                   <property name="属性名" value="简单类型属性值"/>
                   ...
                </bean>

    -->
    <!--简单类型set注入-->
    <bean id="myStudent" class="com.bjpowernode.ba01.Student">
        <property name="name" value="李四"/><!--setName("李四")-->
        <property name="age" value="22" /><!--setAge(22)-->
        <property name="email" value="lisi@qq.com" /><!--setEmail("xxx)-->
    </bean>

    <!--声明日期类-->
    <bean id="mydate" class="java.util.Date">
        <property name="time" value="933295249276"/><!--setTime()-->
    </bean>
</beans>
```

**通过ClassPathXmlApplicationContext实例化ApplicationContext并获取Bean**

```java
String config="applicationContext.xml";
ApplicationContext ctx  = new ClassPathXmlApplicationContext(config);
Student student = (Student) ctx.getBean("myStudent");
System.out.println("student="+student);
Date date = (Date) ctx.getBean("mydate");
System.out.println("date==="+date);
```



### 1.2.2 Spring IoC容器的初始化和依赖注入

### 1.2.3 Spring Bean的生命周期



# 2. 装配Spring Bean

## 2.1依赖注入的三种方式

**注意依赖注入不等于XML配置，下面用XML方式只是举例注入类型，例如注解也能用setter注入，依赖注入的三种方式只是设计原理，装配Bean的时候用的不同的方法都是基于这三种依赖注入方式的**

+ 构造器注入

  Spring采用反射的方式通过使用类的构造方法来完成注入，这就是构造器注入的原理

  ```java
  package com.ssm.chapter10.pojo;
  
  public class Role {
  	private Long id;
  	private String roleName;
  	private String note;
      
  	public Role(String roleName, String note) {
  		this.roleName = roleName;
  		this.note = note;
  	}
  	/****setter and getter****/
  
  }
  ```

  ```xml
  <bean id="role1" class="com.ssm.chapter10.pojo.Role">
      <constructor-arg index="0" value="总经理"/>
      <constructor-arg index="1" value="公司管理者"/>
  </bean>
  ```

  constructor-arg元素用于定义类构造方法的参数，index用于 定义参数的位置，value则是设置值

  参数比较多的时候，构造器注入的可读性比较差，这时候应该考虑使用setter注入

+ setter注入

  setter注入的原理是通过反射调用类的无参构造方法得到一个对象（因此这个类必须有一个无参的构造方法），然后再通过set方法往对象中注入值。

  ```java
  package com.ssm.chapter10.pojo;
  
  public class Role {
  	private Long id;
  	private String roleName;
  	private String note;
  
  	public Role() {
  	}
  
  	public Role(String roleName, String note) {
  		this.roleName = roleName;
  		this.note = note;
  	}
  	/****setter and getter****/
  
  }
  ```

  ```xml
  <bean id="role2" class="com.ssm.chapter10.pojo.Role">
      <property name="roleName" value="高级工程师"/>
      <property name="note" value="重要人员"/>
  </bean>
  ```

+ 接口注入

  有时候有些资源并非来源于自身系统，而是来源于外界，这个时候使用接口注入（留个坑，详细信息请google）

  

## 2.2 装配Bean

Spring中提供了3种方法进行配置Bean，在实际开发中，这三种配置方式都会用到，并且常常混用，因此必须有一个优先级，其优先级从上到下依次递增

+ 在XML中显示配置
+ 在Java的接口和类中实现配置
+ 隐式Bean的发现机制和自动装配原则

第二种和第三种方式对应的就是**注解**的方式

**通常，当配置的类是自己正在开发的工程，那么应该考虑Java配置为主（即2、3两种，优先用3）；如果配置的类并不是自己开发的，建议用1**

### 2.2.1 在XML中显示配置

```java
public class ComplexAssembly{
    private Long id;
    private List<String> list;
    private Map<String,String> map;
    private Properties props;
    private Set<String> set;
    private String[] array;
    /****setter and getter****/
}
```



```xml
<?xml version='1.0' encoding='UTF-8' ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">
    <!--很明显这里的装配用的 是构造器注入的方式-->
    <!--简易装配，用于装配基础类型和String类型-->
	<bean id="role_1" class="com.ssm.chapter9.pojo.Role">
		<property name="roleName" value="高级工程师" />
		<property name="note" value="重要人员" />
	</bean>

	<bean id="role_2" class="com.ssm.chapter9.pojo.Role">
		<property name="id" value="1" />
		<property name="roleName" value="高级工程师" />
		<property name="note" value="重要人员" />
	</bean>
	<bean id="source" class="com.ssm.chapter9.pojo.Source">
		<property name="fruit" value="橙汁" />
		<property name="sugar" value="少糖" />
		<property name="size" value="大杯" />
	</bean>

    <!--ref用于装配引用类型，其值是之前定义的Bean的id-->
	<bean id="juiceMaker2" class="com.ssm.chapter9.pojo.JuiceMaker2">
		<property name="beverageShop" value="贡茶" />
		<property name="source" ref="source" />
	</bean>
	
    <!--装配集合-->
	<bean id="complexAssembly" class="com.ssm.chapter10.pojo.ComplexAssembly">
		<property name="id" value="1" />
		<property name="list">
			<list>
				<value>value-list-1</value>
				<value>value-list-2</value>
				<value>value-list-3</value>
			</list>
		</property>
		<property name="map">
			<map>
				<entry key="key1" value="value-key-1" />
				<entry key="key2" value="value-key-2" />
				<entry key="key3" value="value-key-3" />
			</map>
		</property>
		<property name="props">
			<props>
				<prop key="prop1">value-prop-1</prop>
				<prop key="prop2">value-prop-2</prop>
				<prop key="prop3">value-prop-3</prop>
			</props>
		</property>
		<property name="set">
			<set>
				<value>value-set-1</value>
				<value>value-set-2</value>
				<value>value-set-3</value>
			</set>
		</property>
		<property name="array">
			<array>
				<value>value-array-1</value>
				<value>value-array-2</value>
				<value>value-array-3</value>
			</array>
		</property>
	</bean>
</beans>
```



**如果集合中的元素类型是引用类型（除了String）**

```java
public class UserRoleAssembly{
    private Long id;
    private List<Role> list;
    private Map<Role,User> map;
    private Set<Role> set;
    /****setter and getter****/
}
```



```xml
<?xml version='1.0' encoding='UTF-8' ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

	<bean id="role1" class="com.ssm.chapter10.pojo.Role">
		<property name="id" value="1" />
		<property name="roleName" value="role_name_1" />
		<property name="note" value="role_note_1" />
	</bean>

	<bean id="role2" class="com.ssm.chapter10.pojo.Role">
		<property name="id" value="2" />
		<property name="roleName" value="role_name_2" />
		<property name="note" value="role_note_2" />
	</bean>

	<bean id="user1" class="com.ssm.chapter10.pojo.User">
		<property name="id" value="1" />
		<property name="userName" value="user_name_1" />
		<property name="note" value="role_note_1" />
	</bean>

	<bean id="user2" class="com.ssm.chapter10.pojo.User">
		<property name="id" value="2" />
		<property name="userName" value="user_name_2" />
		<property name="note" value="role_note_1" />
	</bean>

	<bean id="userRoleAssembly" class="com.ssm.chapter10.pojo.UserRoleAssembly">
		<property name="id" value="1" />
		<property name="list">
			<list>
				<ref bean="role1" />
				<ref bean="role2" />
			</list>
		</property>
		<property name="map">
			<map>
				<entry key-ref="role1" value-ref="user1" />
				<entry key-ref="role2" value-ref="user2" />
			</map>
		</property>
		<property name="set">
			<set>
				<ref bean="role1" />
				<ref bean="role2" />
			</set>
		</property>
	</bean>
</beans>
```

如果Bean太多，想放在不同的配置文件之中，可以在多个配置文件中声明Bean，然后在一个总的配置文件中导入其他配置文件即可，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">



    <!--当前是总的文件， 目的是包含其他的多个配置文件， 一般不声明bean
        语法：
        <import resource="classpath:其他文件的路径" />

        classpath:表示类路径。类文件所在的目录， spring通过类路径加载配置文件
    -->
    <!--
    <import resource="classpath:ba06/spring-school.xml" />
    <import resource="classpath:ba06/spring-student.xml" />
    -->


    <!--包含关系的配置文件，可使用通配符（*：表示任意字符）
        注意：总的文件名称，不能包含在通配符范围内（applicationContext.xml不能叫做
        spring-applicationContext.xml）
    -->
    import resource="classpath:ba06/spring-*.xml" /><
</beans>
```



### 2.2.2 通过注解配置Bean

通常不推荐使用XML的方式装配，而推荐使用注解的方式装配Bean.

在Spring中 ，它提供了两种方式来让Spring IoC容器发现Bean:

+ 组件扫描：通过定义资源（即包名）的方式，让Spring IoC容器扫描对应的包，从而把Bean装配进来

```xml
<!--声明组件扫描器：使用注解必须加入这个语句
        component-scan:翻译过来是组件扫描器，组件是java对象。
            属性： base-package 注解在你的项目中的包名。
                  框架会扫描这个包和子包中的所有类，找类中的所有注解。
                  遇到注解后，按照注解表示的功能，去创建对象， 给属性赋值。
-->
<context:component-scan base-package="edu.zju.wht"/>
```

+ 自动装配：通过注解定义，使得 一些依赖关系（ref）可以通过注解完成



**常用的装配Bean的注解**

+ @Component

```java
package com.ssm.chapter10.annotation.pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component(value = "role")
public class Role {
	@Value("1")
	private Long id;
	@Value("role_name_1")
	private String roleName;
	@Value("role_note_1")
	private String note;

	/**setter and getter**/

}

```

@Component代表Spring IoC会把这个类扫描生成Bean实例，其中value属性代表这个类在Spring中的id, 相当于XML方式定义的Bean的id,也可以简写成@Component("role")，甚至可以直接写出@Component，对于不写的，Spring IoC容器就默认类名，但是首字母小写形式作为id, 配置到容器中。

**Bean什么时候会被实例化**

Spring Bean有一个配置选项——lazy-init，默认值是false，Spring IoC会自动初始化Bean。如果这个值设置为true,那么只有当我们使用Spring IoC容器的getBean方法获取它时，它才会进行Bean的初始化。



+ 自动装配@Autowired，用于注入对象

```java
package com.ssm.chapter10.annotation.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.ssm.chapter10.annotation.pojo.Role;
import com.ssm.chapter10.annotation.service.RoleService2;

@Component("RoleService2")
public class RoleServiceImpl2 implements RoleService2 {

	@Autowired
	private Role role = null;

	public Role getRole() {
		return role;
	}

//	@Autowired
	public void setRole(Role role) {
		this.role = role;
	}

	@Override
	public void printRoleInfo() {
		System.out.println("id =" + role.getId());
		System.out.println("roleName =" + role.getRoleName());
		System.out.println("note =" + role.getNote());
	}
}
```

@Autowired注解，表示Spring IoC定位所有的Bean后，这个字段需要按类型注入，这样IoC容器就会寻找资源，然后将其注入。在上面代码的例子中，IoC容器会先生成Role类（这个类的代码上面有）的和RoleService2的实例，然后依据@Autowired注解，按照类型找到定义的实例，在该例子中，即Role类的实例，然后将这个实例注入到RoleService2对象的role属性中。

IoC容器有时候在寻找被注入的实例的时候会寻找失败，默认情况下寻找失败就会抛出异常。但有些情况下，这个属性的注入可有可无，注入不是必须的，有就注入，没有就算了，那么可以通过@Autowired的配置项required来改变它，如下

```xml
@Autowired(required=false)
```

**required的默认值是true**



+ @Primary

@Autowired是Spring提供的默认装配方式，但是有这么一个问题，如果一个接口或者抽象类有多个实现类，并且这个接口或者抽象类在另一个类中作为其属性而存在，那么自动装配的时候，Spring IoC容器怎么知道去找哪一个实现类的实例来装配呢？

因为这种情况下通过类型（ByType）获取的Bean的不唯一，这样自动装配就产生了歧义性。

```java
package com.ssm.chapter10.annotation.service.impl;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

import com.ssm.chapter10.annotation.pojo.Role;
import com.ssm.chapter10.annotation.service.RoleService;

@Component("roleService3")
@Primary
public class RoleServiceImpl3 implements RoleService {
	
	@Override
	public void printRoleInfo(Role role) {
		System.out.print("{id =" + role.getId());
		System.out.print(", roleName =" + role.getRoleName());
		System.out.println(", note =" + role.getNote() + "}");
	}
}
```

如上面所示代码，RoleServiceImpl3也是RoleService接口的实现类，@Primary注解告诉Spring IoC容器，如果存在多个RoleService类型，即RoleService的实现类，那么优先将@Primary注解修饰的实现类的实例注入，这样就可以消除歧义性。

**@Primary只能解决首要性问题，不能解决选择性问题，即不能选择使用接口的具体实现类去注入**



+ @Qualifier

与@Primary注解按照优先性来解决歧义性不同，@Qualifier注解按照名称查找被注入的Bean，这样一来本身就没有歧义性了。

```java
package com.ssm.chapter10.annotation.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import com.ssm.chapter10.annotation.pojo.Role;
import com.ssm.chapter10.annotation.service.RoleService;

@Component
public class RoleController {
	
	@Autowired
	@Qualifier("roleService3")
	private RoleService roleService = null;
	
	public void printRole(Role role) {
		roleService.printRoleInfo(role);
	}
}
```

如上面所示代码，@Qualifier直接指定被注入的Bean的id，即roleService3，即RoleServiceImpl3类的Bean对象



**装在带有参数的构造方法类**

@Autowired和@Qulifier这两个注解还能支持参数，（推测：逻辑上来看只能是构造方法的参数和set方法的参数，因为通过反射来依赖注入都是通过这两类方法来注入的），如下所示：

```java
public RoleController2(@Autowired RoleService roleService){
    this.roleService=roleService;
}
```



**使用@Bean装配Bean**

@Component只能注解到类上，不能注解到方法上，对于第三方的jar包，往往没有源码，只有一个jar包，想要使用第三方的Bean，怎么办呢？

在XML配置方法中可以直接在bean的class属性中指定第三方类的全限定名称即可

注解如何实现这一功能呢？

```java
@Bean(name = "dataSource")
public DataSource getDataSource() {
    Properties props = new Properties();
    props.setProperty("driver", "com.mysql.jdbc.Driver");
    props.setProperty("url", "jdbc:mysql://localhost:3306/chapter10");
    props.setProperty("username", "root");
    props.setProperty("password", "123456");
    DataSource dataSource = null;
    try {
        dataSource = BasicDataSourceFactory.createDataSource(props);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return dataSource;
}
```

@Bean可以注解到方法之上，并且将方法返回的对象作为Spring的Bean，存放在IoC容器中。

这里还配置了@Bean的name选项为dataSource,意味着生成的这个DataSource类型的Bean的id是dataSource.



**在自己的工程中所开发的类尽量使用注解方式，对于引入第三方包或者服务的类，尽量使用XML方式，好处是可以尽量减少对三方包或者服务的细节的理解，更加清晰和明朗。**

