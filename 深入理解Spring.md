# 深入理解Spring

[TOC]

Start: 2023/01/16-

Video: [BV1Ft4y1g7Fb](https://www.bilibili.com/video/BV1Ft4y1g7Fb?p=40) p40

## 0x1. 初步

### 1. 为什么使用 Spring

当对于业务进行需求扩展的时候，比如将数据库从MySQL改为psql，不得不修改以前的代码，从而导致破坏以前代码的逻辑，从而需要重新进行测试CICD等流程，违反了OCP开闭原则（对扩展开放，对修改关闭）和DIP依赖反转原则（面向抽象编程）。

```java
class UserService {
    // UserDao 是接口，后面不 new 具体实现类
    private UserDao userDao;
}
```

但是 `userDao` 为 null，程序无法正常运行。就需要**控制反转**(IoC)进行处理，即不在程序中采用硬编码的方式创建方式和维护对象之间关系。

而 Spring 框架实现了控制反转的编程思想，**依赖注入**(DI)就是控制反转中十分著名的实现方式之一。

依赖注入有两种实现方式：Set注入和构造注入方式

```java
class UserService {
    // UserDao 是接口，后面不 new 具体实现类
    private UserDao userDao;
    
    // 构造方法注入
    UserService(UserDao userDao) {
        this.userDao = userDao;
    }
    
    // Set 注入
    public setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

### 2. Spring简介

Spring 含有 8 大模块，并且是非侵入性的（不依赖其他框架）。

Spring 也是一个对象容器，在 Spring 中对象可以称为 Bean，Spring 包含并且管理对象的配置和生命周期，可以创建单个或者多个的实例。

添加依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.4</version>
</dependency>
```

在 `resources` 目录下创建一个 bean 的xml文件夹，开始配置 bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userBean" class="com.yz.beans.User"/>
</beans>
```

其中 bean 有两个重要的属性：`id` 和 `class`，`id` 是 bean 的唯一标识符，`class` 是 bean 的类名。

通过 xml 文件创建 bean 对象并且使用：

1. 首先需要创建 spring 容器，通过创建 xml 对应的对象 `ClassPathXmlApplicationContext` 并且指定配置文件在资源目录下的路径
2. 使用 `getBean` 方法获取配置文件中的 id 得到对象。

```java
public void testA() {
    // 1. 首先需要创建容器，启动 spring
    ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
    // 2. 获取对象
    Object userBean = context.getBean("userBean");
    System.out.println(userBean);
}
```

Spring 是如何创建对象的？ Spring 是通过反射机制来创建对象的，并且 Bean 的存储是存放在一个 Map 集合中，key 即为 id。

也可以创建 JDK 自带的对象：

```xml
<bean id="date" class="java.util.Date"/>
```

对应 Java 代码：

```java
public void testDate() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("spring.xml");
    Date date = ctx.getBean("date", Date.class);
    System.out.println(date);
}
```

如果对应 id 不存在，程序直接报错。

`ApplicationContext` 接口的父接口是 `BeanFactory`，是一个能生产 bean 的工厂对象，Spring 的 IoC 容器底层使用了工厂模式，底层实现就是 XML 解析 + 工厂模式 + 反射机制。

Spring 在初始化容器的时候就会创建对象，而不是在获取对象的时候。

## 0x2 控制反转 IoC

### 1. 简单注入

🔵注入方式：

1. set 注入，必须要有 set 方法

   对象文件：

   ```java
   public class User {
       int age;
       String name;
       Dog dog;
   
       public void setDog(Dog dog) {
           this.dog = dog;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   }
   ```

   xml 文件：

   ```xml
   <bean id="userBean" class="com.yz.beans.User">
       <property name="age" value="12"/>
       <property name="name" value="小萌"/>
       <property name="dog" ref="dogBean"/>
   </bean>
   ```

2. 构造注入，必须要有对应的构造方法

   索引和参数必须对应

   ```xml
   <bean id="userJackBean" class="com.yz.beans.User">
       <constructor-arg index="0" value="22"/>
       <constructor-arg index="1" value="Jack"/>
       <constructor-arg index="2" ref="dogBean"/>
   </bean>
   ```

对象类型使用 `ref` 进行引用，简答类型使用 `value` 进行设置。

Spring 注入中的简单类型：基本类型+包装类(`int`, `Integer`)、枚举、CharSequence、Number类、Date、Temporal、URI、URL、Locale、Class类。

🔵数组注入

简单类型数组注入：

```xml
<!-- String[] -->
<bean id="dogBean" class="com.yz.beans.Dog">
    <property name="actions">
        <array>
            <value>eat</value>
            <value>shit</value>
            <value>run</value>
        </array>
    </property>
</bean>
```

复杂类型注入：

```xml
<!-- Person[] -->
<bean id="dogBean" class="com.yz.beans.Dog">
    <property name="actions">
        <array>
            <ref bean="b1"/>
            <ref bean="b2"/>
            <ref bean="b3"/>
        </array>
    </property>
</bean>
```

List / Set 注入：将 `array` 标签改为 `list` / `set` 即可

```xml
<!-- List / Set -->
<bean id="dogBean" class="com.yz.beans.Dog">
    <property name="actions">
        <list>
            <value>eat</value>
            <value>shit</value>
            <value>run</value>
        <list>
    </property>
</bean>
```

Map 注入：

```xml
<!-- Map -->
<bean id="dogBean" class="com.yz.beans.Dog">
    <property name="actions">
        <map>
            <entry key="k" value="v"/>
            <entry key-ref="k" value-ref="v"/>
        </map>
    </property>
</bean>
```

Properties 注入：

```xml
<!-- Map -->
<bean id="dogBean" class="com.yz.beans.Dog">
    <property name="actions">
        <props>
            <prop key="k">v</prop>
        </props>
    </property>
</bean>
```

null 和 空字符串注入：配置中不给属性就是 `null`，或者使用 `<null/>` 标签

```xml
<property name="age">
	<null/>
</property>
<!-- 空字符串 -->
<property name="name" value="">
```

### 2. 命名空间注入

🔵p 命名空间注入：基于 set 方法

在 beans 标签属性中添加 `xmlns:p="http://www.springframework.org/schema/p"`

```xml
<bean id="dogBean" class="com.yz.beans.Dog"/>
<bean id="userBean" class="com.yz.beans.User" p:age="18" p:dog-ref="dogBean"/>
```

🔵c 命名空间注入：

基于构造方法，用于简化构造注入

在 beans 标签属性中添加 `xmlns:c="http://www.springframework.org/schema/c"`

```xml
<bean id="dogBean" class="com.yz.beans.Dog"/>
<bean id="userBean" class="com.yz.beans.User" c:_0 ="18" c:dog-ref="dogBean"/>
```

### 3. 自动装配

> 基于 set 方法

Spring 还可以完成自动化注入即自动装配(autowire)，可以根据名称进行装配，也可以根据类名进行装配。

ByName：被装配的 bean 必须和 set 后的名字一致

```xml
<bean id="dog" class="com.yz.beans.Dog"/>
<bean id="userBean" class="com.yz.beans.User" autowire="byName">
    <!-- 简单类型需要指定 -->
    <property name="age">18</property>
    <!-- 复杂类型 Dog 则根据 autowire 自动绑定 -->
</bean>
```

ByType 根据类型进行注入：id 可以任意指定，只要存在对应类型的 bean 即可进行绑定

```xml
<bean id="dog111" class="com.yz.beans.Dog"/>
<bean id="userBean" class="com.yz.beans.User" autowire="byType">
    <!-- 简单类型需要指定 -->
    <property name="age">18</property>
    <!-- 复杂类型 Dog 则根据 autowire 自动绑定 -->
</bean>
```

但是 ByType 的对应类型的实例**只能有一个**！

### 4. 多配置文件

为了防止配置文件过大，读取时间过长。

一般分为主配置文件和子配置文件。主配置文件是用来包含其他配置文件，不定义对象，用于导入其他配置文件。

```
配置文件目录
ba04
	spring-total.xml
	spring-student.xml
	spring-school.xml
```

student和schoold的xml文件只保存自己的bean，类似上述的配置。

在total主配置文件中进行配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--   classpath需要在target目录下的classes进行寻找 -->

    <import resource="classpath:ba04/spring-student.xml"/>
    <import resource="classpath:ba04/spring-school.xml"/>
    
    <!-- 可以进行包含关系进行通配符匹配 -->
    <!-- 但是主配置文件不能与这个文件名匹配，比如不可以是spring-total.xml  -->
    <import resource="classpath:ba04/spring-*.xml"/>
</beans>
```

### 5. 引入 properties 文件

在 beans 属性中添加 `xmlns:context="http://www.springframework.org/schema/context"`，以及 `http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd`

使用 `context:property-placeholder` 引入对应的 properties 文件，

```properties
# user.properties
user.habit=aka.dropilin
user.age=22
```

xml 文件为：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:property-placeholder location="user.properties"/>
    <bean id="u" class="com.yz.beans.User">
        <property name="name" value="${user.habit}"/>
        <property name="age" value="${user.age}"/>
    </bean>

</beans>
```

## 0x3 Beans 对象

### 1. Bean 作用域

使用两个 `getBean` 来获取对象

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("spring-scope.xml");
User u = ac.getBean("user", User.class);
User u2 = ac.getBean("user", User.class);
System.out.println(u);
System.out.println(u2);
```

输出：

```
com.yz.beans.User@3eb738bb
com.yz.beans.User@3eb738bb
```

两个获取的都是同一个对象。

Spring 默认情况下是单例的，在 Spring 初始化的时候就初始化了，每次返回的都是同一个对象。

当将 `scope` 设置为 `prototype` 就可以进行原型对象创建多例，默认为 `singleton` 为单例，在 Spring 初始化的时候不会进行创建对象，只有在使用的时候才会进行创建。

> scope 还支持 `request` 和 `session` 等的值，当为 web 项目引入 springmvc 的时候就可以使用
>
> * request：一次请求中对应一个 bean
> * session：一次会话对应一个 bean
> * application：一个应用对应一个 bean
> * websocket：一个 ws 生命周期对应一个 bean

```xml
<bean id="user" class="com.yz.beans.User" scope="prototype">
    <property name="name" value="小俊"/>
    <property name="age" value="18"/>
</bean>
```

输出：

```
com.yz.beans.User@31190526
com.yz.beans.User@662ac478
```

🔵自定义 scope

还支持自定义 scope，这里演示一个线程对应一个 bean

```xml
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
    <property name="scopes">
        <!--   将对应的 threadScope 注册到 scope 中   -->
        <map>
            <entry key="threadScope">
                <bean class="org.springframework.context.support.SimpleThreadScope"/>
            </entry>
        </map>
    </property>
</bean>

<bean id="user" class="com.yz.beans.User" scope="threadScope">
    <property name="name" value="小俊"/>
    <property name="age" value="18"/>
</bean>
```

对应的 Java 代码：

```java
public void testA() {
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-scope.xml");
    System.out.println(ac.getBean("user", User.class));
    System.out.println(ac.getBean("user", User.class));
    new Thread(() -> {
        System.out.println(ac.getBean("user", User.class));
        System.out.println(ac.getBean("user", User.class));
    }).start();
}
```

输出：

```
com.yz.beans.User@55b0dcab
com.yz.beans.User@55b0dcab
com.yz.beans.User@6ec5e77d
com.yz.beans.User@6ec5e77d
```

