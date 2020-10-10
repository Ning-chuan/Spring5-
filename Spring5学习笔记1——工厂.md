# Spring5学习笔记1——工厂

[TOC]

## 第一部分：Spring工厂

### 第一章、引言

#### 1. EJB存在的问题

>  **EJB (Enterprise Java Beans)** 是基于分布式事务处理的企业级应用程序的组件。Sun公司发布的文档中对EJB的定义是：EJB是用于开发和部署多层结构的、分布式的、面向对象的Java应用系统的跨平台的构件体系结构。

EJB是一个**重量级**框架：

- 运行环境苛刻
- 代码移植性差

#### 2. 什么是Spring

Spring是一个**轻量级**的javaEE解决方案，整合了众多优秀的设计模式。

- 轻量级体现在哪里？
  1. 对运行环境没有额外要求
  2. 代码移植性高，不需要实现额外接口。
- JavaEE解决方案：

<img src="https://i.loli.net/2020/09/27/M3uwTsUNvpgQ9RZ.png" alt="image-20200927003815440" style="zoom:67%;" />

- Spring整合的设计模式：

  ```markdown
  1. 工厂模式
  2. 代理模式
  3. 模板模式
  4. 策略模式
  ```

#### 3. 什么是设计模式

```markdown
1.广义概念：
	面向对象设计中，解决特定问题的经典代码。
2.狭义概念：
	GOF4人帮定义的23种设计模式：
		工厂模式、抽象工厂模式、单例模式、建造者模式、原型模式、设配器模式、桥接模式、过滤器模式、组合模式、装饰者模式、外观模式、享元模式、代理模式、责任链模式、命令模式、解释器模式、迭代器模式、中介者模式、备忘录模式、观察者模式、状态模式、空对象模式、策略模式、模板模式、访问者模式
```

#### 4. 工厂设计模式

##### 4.1 什么是工厂设计模式

```markdown
1. 概念：创建对象交给工厂，而不是自己new
2. 优势：解耦合
	耦合：指代码间的强关联关系，一方的改变会影响另一方。
	问题：不利于代码的维护。
		User user = new User();
```

##### 4.2 简单工厂的设计 

```java
package com.yuziyan.basic;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class BeanFactory {

    private static Properties env = new Properties();
    static{
        try {
            //第一步 获得IO输入流
            InputStream inputStream = BeanFactory.class.getResourceAsStream("applicationContext.properties");
            //第二步 文件内容 封装 Properties集合中 key = userService value = com.yuziyan.UserServiceImpl
            env.load(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /*
	   对象的创建方式：
	       1. 直接调用构造方法 创建对象  UserService userService = new UserServiceImpl();
	       2. 通过反射的形式 创建对象 解耦合
	       Class clazz = Class.forName("com.yuziyan.basic.UserServiceImpl");
	       UserService userService = (UserService)clazz.newInstance();
     */

    public static UserService getUserService(){
        UserService userService = null;
        try {
            //                          com.yuziyan.basic.UserServiceImpl
            Class clazz = Class.forName(env.getProperty("userService"));
            userService = (UserService) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return userService;
    }

    public static UserDao getUserDao(){
        UserDao userDao = null;
        try {
            Class clazz = Class.forName(env.getProperty("userDao"));
            userDao = (UserDao) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return userDao;
    }
}
```

自定义配置文件applicationContext.properties:

```properties
# 自定义properties文件，存储要管理的类全名
# 在java代码中用对应的Properties集合 来读取这个文件的内容
# Properties是专门处理属性文件的类，以键值对的形式存储
# Properties [userService = com.yuziyan.xxx.UserServiceImpl]
# Properties.getProperty("userService")

userDao=com.yuziyan.basic.UserDaoImpl
userService=com.yuziyan.basic.UserServiceImpl
```

简单工厂存的问题：存在大量的==代码冗余==

<img src="https://i.loli.net/2020/09/27/1jgesOBqzVyn7Ab.png" alt="image-20200927114116886" style="zoom:80%;" />

##### 4.3 通用工厂的设计

- 通用工厂的代码：

```java
package com.yuziyan.basic;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class BeanFactory {

    private static Properties env = new Properties();
    static{
        try {
            //第一步 获得IO输入流
            InputStream inputStream = BeanFactory.class.getResourceAsStream("applicationContext.properties");
            //第二步 文件内容 封装 Properties集合中 key = userService value = com.yuziyan.UserServiceImpl
            env.load(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    //通用工厂
    public static Object getBean(String key){
        Object res = null;
        try {
            Class clazz = Class.forName(env.getProperty(key));
            res = clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return res;
    }
}

```

- 通用工厂的使用方式：

```markdown
1. 定义类型 (类)
2. 通过配置文件的配置告知工厂(applicationContext.properties)
   key = value
3. 通过工厂获得类的对象
   Object ret = BeanFactory.getBean("key")
```

#### 5. 总结

**Spring本质：**工厂 ApplicationContext (applicationContext.xml)

### 第二章、第一个Spring程序

#### 1. 软件版本

```markdown
1. JDK1.8+
2. Maven3.5+
3. IDEA2018+
4. SpringFramework 5.1.4 
   官方网站 www.spring.io
```

#### 2. 环境搭建

- Spring的jar包（在pom.xml中加入依赖）

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>5.1.4.RELEASE</version>
</dependency>
```

- Spring的配置文件

```markdown
1. 配置文件的放置位置：任意位置，没有硬性要求
2. 配置文件的命名：没有硬性要求，但建议：applicationContext.xml

思考：日后应用Spring框架时，需要进行配置文件路径的设置。
```

#### 3. Spring的核心API

**ApplicationContext**

```markdown
作用：Spring提供的ApplicationContext这个工厂，用于对象的创建
好处：解耦合
```

- 特点：

```markdown
ApplicationContext是接口类型，屏蔽了实现的差异。
非web环境 ： ClassPathXmlApplicationContext (main junit)
web环境  ：  XmlWebApplicationContext
```

![image-20200927121905306](https://i.loli.net/2020/09/27/sgB1CiNVhMvayrw.png)

- 重量级资源

```markdown
ApplicationContext工厂的对象占用大量内存。
不会频繁的创建对象 ： 一个应用只会创建一个工厂对象。
ApplicationContext工厂：一定是线程安全的(多线程并发访问)
```

#### 4. 程序开发

```markdown
1. 创建类型
2. 配置文件的配置 applicationContext.xml
   <bean id="person" class="com.yuziyan.basic.Person"/>
3. 通过工厂类，获得对象
   ApplicationContext
          |- ClassPathXmlApplicationContext 
   ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
   Person person = (Person)ctx.getBean("person");
```

#### 5. 细节分析

- 名词解释

  ```mark
  Spring工厂创建的对象，叫做bean或者组件(component)
  ```

- Spring工厂的相关方法

  ```java
  //这种方式获取对象，不需要强制类型转换
  Person person = ctx.getBean("person", Person.class);
  System.out.println("person = " + person);
  
  //当前Spring的配置文件中 只能有一个<bean class是Person类型
  Person person1 = ctx.getBean(Person.class);
  System.out.println("person1 = " + person1);
  
  //获取配置文件中所有bean标签的id值  person person1
  String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
  for (String beanDefinitionName : beanDefinitionNames) {
  System.out.println("beanDefinitionName = " + beanDefinitionName);
  }
  
  //根据类型获取配置文件中对应bean标签的id值
  String[] beanNamesForType = ctx.getBeanNamesForType(Person.class);
  for (String id : beanNamesForType) {
  System.out.println("id = " + id);
  }
  
  //用于判断是否存在指定id值的bean，不能判断name值
  System.out.println(ctx.containsBeanDefinition("a"));
  
  //用于判断是否存在指定id值的bean，可以判断name值
  System.out.println(ctx.containsBean("person"));
  ```

- 配置文件中需要注意的细节

  1. 只配置class属性

      ```xml
     <bean  class="com.yuziyan.basic.Person"/>`
     ```

     - 上述这种配置，没有指定id，Spring会自动生成一个 id，`com.yuziyan.basic.Person#0`，可以使用 `getBeanNamesForType()` 等方法验证。
       
  -  应用场景：
     
    ​		如果这个bean只需要使用一次，那么就可以省略id值
       ​        如果这个bean会使用多次，或者被其他bean引用则需要设置id值
    
  2. name属性

      - 作用：用于在Spring的配置文件中，为bean对象定义别名(小名)

      - name与id的相同点：

        1. `ctx.getBean("id")` 或 `ctx.getBean("name")` 都可以创建对象；
        2. `<bean id="person" class="Person"/>` 与 `<bean name="person" class="Person"/>` 等效；

      - name与id的不同点：

        1. 别名可以定义多个,但是 id 属性只能有⼀个值；

        2. XML 的 id 属性的值，命名要求：必须以字⺟开头，可以包含 字⺟、数字、下划线、连字符；不能以特殊字符开头 `/person`；

           XML 的 name 属性的值，命名没有要求，`/person` 可以。
           但其实 XML 发展到了今天：ID属性的限制已经不存在，`/person`也可以。
#### 6. Spring工厂的底层实现原理(简易版)

![image-20200415113032782](https://i.loli.net/2020/09/27/l7eNtyKMHT2Zsmn.png)

#### 7. 思考

问题：未来在开发过程中，是不是所有的对象，都会交给 Spring ⼯⼚来创建呢？

回答：理论上是的，但是有特例 ：**实体对象(entity)** 是不会交给Spring创建，它由持久层框架进⾏创建。

### 第三章、Spring5.x与日志框架的整合

#### 1.为什么要整合日志框架？

Spring与日志框架进行整合，日志框架就可以在控制台中，输出Spring框架运行过程中的一些重要的信息。
好处：便于了解Spring框架的运行过程，利于程序的调试

> 默认日志框架
> Spring 1.x、2.x、3.x 早期都是基于`commonslogging.jar`
> Spring 5.x 默认整合的⽇志框架 logback、log4j2

#### 2.Spring如何整合日志框架？

以Spring5.x整合log4j 为例

1. `pom.xml`文件添加`log4j`依赖：[^相当于导入了log4j.jar包]

   ```xml
   <dependency>
     <groupId>org.slf4j</groupId>
     <artifactId>slf4j-log4j12</artifactId>
     <version>1.7.25</version>
   </dependency>
   <dependency>
     <groupId>log4j</groupId>
     <artifactId>log4j</artifactId>
     <version>1.2.17</version>
   </dependency>
   ```

2. 引⼊ `log4.properties` 配置⽂件：

   ```properties
   # resources文件夹根目录下
   ### 配置根
   log4j.rootLogger = debug,console
   
   ### 日志输出到控制台显示
   log4j.appender.console=org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target=System.out
   log4j.appender.console.layout=org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
   ```

### 第四章、注入( injection)

#### 1. 什么是注入？

> 通过Spring工厂及配置文件，为所创建对象的成员变量赋值

#### 2. 为什么要注入？

- 通过编码的方式，为成员变量进行赋值，存在耦合

- 注入的好处：**解耦合**

  ~~~java
  public void test3(){
  	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  	Person person = (Person) ctx.getBean("person");
  	 //通过代码为变量赋值, 存在耦合, 如果以后想修改变量的值, 需要修改代码, 重新编译
  	person.setName("于自言");
  	person.setAge(16);
  	System.out.println("person = " + person);
  }
  ~~~

#### 3. 如何注入？

- 前提：类的成员变量提供**set get方法**

- 配置Spring配置文件

  ```xml
  <bean id="person1" name="p" class="com.yuziyan.basic.Person">
      <property name="name">
          <value>张无忌</value>
      </property>
      <property name="age">
          <value>200</value>
      </property>
  </bean>
  ```

#### 4. Spring注入的原理分析（简易版）

**Spring通过底层调用对象属性对应的set方法，完成成员变量的赋值，这种方式我们也称之为set注入**

![image-20200415191157364](https://i.loli.net/2020/09/27/F3uTJ1vU9z5xeZ6.png)



### 第五章、Set注入详解

针对于不同类型的成员变量，需要在`<property>`标签中嵌套其他标签：

```xml
<property>
	xxxxx
</property>
```

![image-20200416090518713](https://i.loli.net/2020/09/27/Eo8AnxPFYa3VuRL.png)

#### 1. JDK内置类型

##### 1.1 String+8中基本类型

```xml
<!-- value标签 -->
<property name="name">
    <value>张无忌</value>
</property>
<property name="age">
    <value>200</value>
</property>
```

##### 1.2 数组

```xml
<!-- list标签 -->
<property name="emails">
    <list>
        <value>1234@qq.com</value>
        <value>yangGuo@qq.com</value>
        <value>baiXiaoSheng@qq.com</value>
        <value>Trump@qq.com</value>
    </list>
</property>
```

##### 1.3 Set集合

~~~xml
<!-- set标签 -->
<property name="tels">
    <set>
        <value>131********</value>
        <value>159********</value>
        <value>176********</value>
    </set>
</property>
<!-- Set集合中有非基本类型时： -->
<set>
   <ref bean
   <set 
</set>
~~~

##### 1.4 List集合

```xml
<!-- list标签 -->
<property name="address">
    <list>
        <value>花果山水帘洞</value>
        <value>铁岭</value>
        <value>东土大唐</value>
    </list>
</property>
<!-- List集合中有非基本类型时： -->
<list>
   <ref bean
   <set 
</list>
```

##### 1.5 Map集合

```xml
<!-- 需要用到map entry key三个标签 -->
<property name="qqs">
    <map>
        <entry>
            <key><value>周芷若</value></key>
            <value>备胎</value>
        </entry>
        <entry>
            <key><value>赵敏</value></key>
            <value>爱人</value>
        </entry>
    </map>
</property>
<!-- Map集合中有非基本类型时： -->
<entry>
	<key><value>chenyn</value></key>
	<ref bean
</entry>
```

##### 1.6 Properties

```xml
<!-- props和prop标签 -->
<property name="p">
    <props>
        <prop key="唐僧">白骨精</prop>
        <prop key="Tom">Jerry</prop>
    </props>
</property>
```

##### 1.7 复杂的JDK类型（比如Date）

```markdown
需要自定义类型转换器处理
```

#### 2. 用户自定义类型

##### 2.1 方式一

- 为成员变量提供get set方法

- 配置文件中赋值（注入）

  ![image-20200927175058105](https://i.loli.net/2020/09/27/tWViEPrpzalGLXs.png)

##### 2.2 方式二

> 方式一存在的问题：
>
> 1. 配置文件代码冗余。
> 2. 被注入的对象(UserDAO),多次创建，浪费（JVM)内存资源。

- 为成员变量提供set get方法

- 配置文件中进行配置

  ![image-20200927174935295](https://i.loli.net/2020/09/27/szDiZFg1WXSHlbr.png)

**注意：**Spring4.x 废除了 `<ref local=""/>` 基本等效 `<ref bean=""/>`；

#### 3. Set注入的简化写法

##### 3.1 基本属性简化

- JDK类型注入：（注意：`value`属性 只能简化 8种基本类型+String 注入标签）：

  ```xml
  <!-- 简化前 -->
  <bean id="person2" class="com.yuziyan.basic.Person">
      <property name="name">
          <value>韩信</value>
      </property>
  </bean>
  
  <!-- 简化后 -->
  <bean id="person2" class="com.yuziyan.basic.Person">
      <property name="name" value="韩信"/>
  </bean>
  ```

- 用户自定义类型注入：

  ```xml
  <!-- 简化前 -->
  <bean id="userDao" class="com.yuziyan.basic.UserDaoImpl"/>
  <bean id="userService" class="com.yuziyan.basic.UserServiceImpl">
      <property name="userDao">
          <ref bean="userDao"/>
      </property>
  </bean>
  
  <!-- 简化后 -->
  <bean id="userDao" class="com.yuziyan.basic.UserDaoImpl"/>
  <bean id="userService" class="com.yuziyan.basic.UserServiceImpl">
      <property name="userDao" ref="userDao"/>
  </bean>
  ```

##### 3.2 基于p命名空间的简化

- JDK类型：

  ```xml
  <!-- 简化前：-->
  <bean id="person3" class="com.yuziyan.basic.Person">
      <property name="name">
          <value>沙和尚</value>
      </property>
  </bean>
  
  <!-- 简化后：-->
  <bean id="person3" class="com.yuziyan.basic.Person" p:name="沙和尚"/>
  ```

- 用户自定义类型：

  ```xml
  <!-- 简化前：-->
  <bean id="userDao" class="com.yuziyan.basic.UserDaoImpl"/>
  <bean id="userService" class="com.yuziyan.basic.UserServiceImpl">
      <property name="userDao">
          <ref bean="userDao"/>
      </property>
  </bean>
  
  <!-- 简化后：-->
  <bean id="userService1" class="com.yuziyan.basic.UserServiceImpl" p:userDao-ref="userDao"/>
  ```

### 第六章、构造注入

> **构造注入**：Spring解析配置文件，==调用构造方法==，为成员变量赋值。

#### 1. 开发步骤

- 提供带参数构造方法

  ```java
  public class Customer implements Serializable {
      private String name;
      private int age;
  	//带参构造方法：
      public Customer(String name, int age) {
          this.name = name;
          this.age = age;
      }
  }
  ```

- Spring配置文件中赋值（注入）

  ```xml
  <bean id="customer" class="com.yuziyan.basic.constructor.Customer">
      <constructor-arg>
          <value>武松</value>
      </constructor-arg>
      <constructor-arg>
          <value>234</value>
      </constructor-arg>
  </bean>
  ```

#### 2. 构造方法重载

##### 2.1 参数个数不同时

```markdown
通过控制<constructor-arg>标签的数量进行区分
```

##### 2.2 参数个数相同时

```markdow
通过在标签引入 type属性 进行类型的区分 <constructor-arg type="">
```

#### 3. 注入总结

未来的实战中，应用set注入还是构造注入？

答案：==set注入==更多，原因如下：

1. 构造注入麻烦 (重载)。
2. Spring框架底层大量应用了 set注入。

![image-20200416155620897](https://i.loli.net/2020/09/27/bakfwOJ8oPzxcXK.png)

### 第七章、控制反转 与 依赖注入

#### 1. 控制反转(IOC Inverse of Control)

- 含义：**把对于成员变量赋值的控制权，从代码中反转(转移)到Spring工厂和配置文件中完成。**
- 好处：解耦合
- 底层实现：工厂设计模式

![image-20200416161127972](https://i.loli.net/2020/09/27/u6EThScYsMyl2fL.png)

#### 2. 依赖注入(DI Dependency Injection )

- 含义：**当一个类需要另一个类时，就意味着依赖，这时可以把另一个类作为本类的成员变量，最终通过Spring配置文件进行注入(赋值)。**
- 好处：解耦合。

![image-20200416162615816](https://i.loli.net/2020/09/27/fi9MkdNV3IwR1We.png)

### 第八章、Spring工厂创建复杂对象

![image-20200416164044047](https://i.loli.net/2020/09/28/JEgoFqI4wduZSea.png)

#### 1. 什么是复杂对象

- 含义：指的就是不能直接通过new构造方法创建的对象。

- 举例：Connection

  ​			SqlSessionFactory

#### 2. Spring工厂创建复杂对象的3种方式

##### 2.1 FactoryBean接口

- 开发步骤

  1. 实现FactoryBean接口

     ![](https://i.loli.net/2020/09/28/nGakRV79iOe4PqK.png)

  2. Spring配置文件中注册

     ```xml
     <bean id="conn" class="com.yuziyan.factorybean.ConnectionFactoryBean"/>
     ```

     注意：

     - 如果class类型是FactoryBean接口的实现类，那么通过id值获得的是这个类`getObject()`方法所返回的对象。比如 Connection SqlSessionFactory

     - 由于我们此时想获取的是Connection对象，所以需要在pom.xml文件中加入相关的依赖

       ```xml
       <!-- MySql连接 -->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.48</version>
       </dependency>
       ```

- 细节分析

  1. 如果就想获得FactoryBean类型的对象，在id前加上&符号，`ctx.getBean("&conn")`，此时获得的就是ConnectionFactoryBean对象本身。

  2. `isSingleton()`方法，返回  true 只会创建一个复杂对象，返回 false 每一次都会创建新的对象

     问题：根据这个对象的特点 ，决定是返回true (SqlSessionFactory) 还是 false  (Connection)。

  3. mysql高版本连接创建时，需要制定SSL证书，解决问题的方式。

     ~~~markdown
     url = "jdbc:mysql://localhost:3306/suns?useSSL=false"
     ~~~

  4. 体会依赖注入(DI)

     可以把ConnectionFactoryBean中依赖的4个字符串信息 ，在配置文件中进行注入 ，解耦合。

     ```xml
     <bean id="conn" class="com.yuziyan.factorybean.ConnectionFactoryBean">
         <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
         <property name="url" value="jdbc:mysql://localhost:3306/test?useSSL=false"/>
         <property name="username" value="root"/>
         <property name="password" value="root"/>
     </bean>
     ```

- FactoryBean的实现原理[简易版]

  Spring内部运行流程：

  1. 通过`conn`获得ConnectionFactoryBean类的对象。
  2. 进而通过instanceof 判断是否是FactoryBean接口的实现类。
  3. 如果是，Spring按照规定调用`getObject()`方法返回Connection类的对象。体现了==接口回调==的特点。

  ![image-20200417114723005](https://i.loli.net/2020/09/28/cq3UObdpVIXSLYn.png)

- FactoryBean总结

  Spring中用于创建复杂对象的一种方式，也是Spring原生提供的，后续讲解Spring整合其他框架，大量应用FactoryBean

##### 2.2 实例工厂

- 实例工厂的作用：

  1. 避免Spring框架的侵入 
  2. 整合遗留系统 

- 开发步骤：

  1. 定义实例工厂类：

     ```java
     public class ConnectionFactory {
         public Connection getConnection(){
             //xxxx
             return conn;
         }
     }
     ```

  2. Spring配置文件中注册：

     ```xml
     <bean id="connFactory" class="com.yuziyan.factorybean.ConnectionFactory"/>
     
     <bean id="conn" factory-bean="connFactory" factory-method="getConnection"/>
     ```

##### 2.3 静态工厂

- 静态工厂的作用（同实例工厂）：

  1. 避免Spring框架的侵入 
  2. 整合遗留系统 

- 开发步骤：

  1. 定义静态工厂类：

     ```java
     public class StaticConnectionFactory {
         public static Connection getConnection(){
             //xxxx
             return conn;
         }
     }
     ```

  2. Spring配置文件中注册：

     ```xml
     <bean id="conn1" class="xxx.StaticConnectionFactory" factory-method="getConnection"/>
     ```

#### 3. Spring工厂创建对象总结

![image-20200417152030222](https://i.loli.net/2020/09/28/nue5r8N6lsLvKWq.png)

### 第九章、控制Spring工厂创建对象的次数

#### 1.  为什么要控制对象的创建次数？

- ==为了减少不必要的内存浪费。==

- 什么样的对象只创建一次？

  ```markdown
  1. SqlSessionFactory
  2. DAO
  3. Service
  ...
  ```

- 什么样的对象 每一次都要创建新的？

  ```markdown
  1. Connection
  2. SqlSession | Session
  3. Struts2 Action
  ...
  ```

#### 2.  如何控制简单对象的创建次数 

```xml
<bean id="account" scope="singleton|prototype" class="xxxx.Account"/>
```

`singleton`:只会创建一次简单对象 默认值
`prototype`:每一次都会创建新的对象

#### 3.  如何控制复杂对象的创建次数

```java
FactoryBean{
   isSingleton(){
      return true;//只会创建一次
      return false;//每一次都会创建新的
   }

}
//如没有isSingleton方法，还是通过scope属性，进行控制。
```

## 第二部分：Spring工厂的高级特性

### 第十章、对象的生命周期

#### 1.  什么是对象的生命周期

含义：==一个对象创建、存活、消亡的一个完整过程==。

#### 2.  为什么要学习对象的生命周期

由Spring负责对象的创建、存活、销毁，了解生命周期，有利于我们使用好Spring为我们创建的对象。

#### 3.  生命周期的3个阶段

##### 3.1 创建阶段

- `scope="singleton"`

  此时会在创建Spring工厂时，创建对象。

  注意：如果同时设置了`lazy-init="true"`，那么会在获取对象`ctx.getBean("")`时创建。

- `scope="prototype"`

  Spring工厂会在获取对象时，创建对象，即调用`ctx.getBean("")`方法时创建。

==注意：如果有属性需要注入(DI)，会在创建完成时立即进行注入。==

##### 3.2 初始化阶段

***创建阶段完成后，Spring会调用对象的初始化方法，完成对应的初始化操作。***

程序员提供初始化方法的途径：

1. 实现InitializingBean接口：

   ```java
   //实现这个方法，完成初始化操作
   public void afterProperitesSet(){
     
   }
   ```

2. 对象中提供一个普通的方法同时配置Spring配置文件：

   ```java
   public void myInit(){
     
   }
   ```

   ```xml
   <bean id="product" class="xxx.Product" init-method="myInit"/>
   ```

细节分析：

- 如果一个对象即实现InitializingBean，同时又提供的普通的初始化方法 ，顺序：

  ```markdown
  1. InitializingBean 
  2. 普通初始化方法
  ```

- 注入一定发生在初始化操作的前面

- 什么叫做初始化操作：资源的初始化：数据库、IO、网络 .....

##### 3.3销毁阶段

***Spring销毁对象`ctx.close();`前，会调用对象的销毁方法，完成销毁操作***

程序员定义销毁方法的途径：

1. 实现DisposableBean

   ```java
   public void destroy()throws Exception{
     
   }
   ```

2. 定义一个普通的销毁方法同时配置Spring配置文件：

   ```java
   public void myDestroy()throws Exception{
   
   }
   ```

   ```xml
   <bean id="" class="" init-method="" destroy-method="myDestroy"/>
   ```

细节分析：

- 销毁方法的操作只适用于 `scope="singleton"`
- 销毁操作主要指资源的释放操作，比如`io.close();` ` connection.close();`

### 第十一章、配置文件参数化

#### 1. 什么是配置文件参数化？

答：把Spring配置文件中需要经常修改的字符串信息，转移到一个更小的配置文件中。

```markdown
1. Spring的配置文件中存在需要经常修改的字符串？
   存在 以数据库连接相关的参数 代表
2. 经常变化字符串，在Spring的配置文件中，直接修改
   不利于项目维护(修改)
3. 转移到一个小的配置文件(.properties)
   利于维护(修改)
   
配置文件参数化：利于Spring配置文件的维护(修改)
```

#### 2. 配置文件参数化的开发步骤:

##### 2.1 提供一个小的配置文件(.properities)

```properties
# 名字：随便
# 放置位置：随便

jdbc.driverClassName = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/test?useSSL=false
jdbc.username = root
jdbc.password = root
```

##### 2.2 Spring的配置文件与小配置文件进行整合

```xml
<context:property-placeholder location="classpath:/db.properties"/>
```

##### 2.3 在Spring配置文件中通过`${key}`获取小配置文件中对应的值

![image-20200928215903162](https://i.loli.net/2020/09/28/omszDWbR6yZvwLV.png)

### 第十二章、自定义类型转换器

 #### 1. 类型转换器的作用

Spring通过类型转换器把配置文件中字符串类型的数据，转换成对象中成员变量对应类型的数据，进而完成了注入。

![image-20200418201732220](https://i.loli.net/2020/09/28/sF6Xgv7fcGzhMRl.png)

#### 2. 为什么要自定义类型转换器？

原因：实际应用中需要转换某种特定的类型，且Spring内部没有提供这种类型转换器时，需要程序员自己定义。

#### 3. 自定义类型转换器的开发步骤

##### 3.1 实现 `Converter<>`接口

```java
public class MyDateConverter implements Converter<String, Date> {
    /*
    convert方法作用：String --->  Date
                   SimpleDateFormat sdf = new SimpleDateFormat();
                   sdf.parset(String) ---> Date
    param:source 代表的是配置文件中 日期字符串 <value>2020-10-11</value>

    return : 当把转换好的Date作为convert方法的返回值后，Spring自动的为birthday属性进行注入（赋值）

    */
    public Date convert(String source) {
        Date date = null;
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            date = sdf.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

##### 3.2 在Spring的配置文件中进行配置

```xml
<!-- 创建自定义转换器对象 -->
<bean id="myDateConverter" class="com.yuziyan.converter.MyDateConverter"/>

<!-- 在Spring中注册自定义的转换器 -->
<!-- 目的：告知Spring框架，我们所创建的MyDateConverter是一个类型转换器 -->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <ref bean="myDateConverter"/>
        </set>
    </property>
</bean>

<!-- 上面两步完成之后就可以直接使用了 -->
```

#### 4. 细节

##### 4.1 体会依赖注入(DI)：`MyDateConverter`中的日期的格式，可以通过依赖注入的方式，由配置文件完成赋值。

```java
public class MyDateConverter implements Converter<String, Date> {

    private String pattern;
    
    public String getPattern() { return pattern; }
    
    public void setPattern(String pattern) { this.pattern = pattern; }

    /*
        convert方法作用：String --->  Date
                       SimpleDateFormat sdf = new SimpleDateFormat();
                       sdf.parset(String) ---> Date
        param:source 代表的是配置文件中 日期字符串 <value>2020-10-11</value>

        return : 当把转换好的Date作为convert方法的返回值后，Spring自动的为birthday属性进行注入（赋值）

        */
    public Date convert(String source) {
        Date date = null;
        try {
            SimpleDateFormat sdf = new SimpleDateFormat(pattern);
            date = sdf.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

```xml
<!--创建自定义类型转换器对象-->
<bean id="myDateConverter" class="com.yuziyan.converter.MyDateConverter">
    <property name="pattern" value="yyyy-MM-dd"/>
</bean>
```

##### 4.2 `ConversionSeviceFactoryBean` 定义id属性，值必须是 `conversionService `

##### 4.3 Spring框架内置日期类型的转换器

```markdown
日期格式：2020/05/01 (不支持 ：2020-05-01)
```

### 第十三章、后置处理Bean

#### 1. BeanPostProcessor的作用：

==对Spring工厂所创建的对象，进行再加工。==

注意：BeanPostProcessor是接口

#### 2. 后置处理Bean的运行原理分析

![image-20200420155053027](https://i.loli.net/2020/10/06/Sql17ei5gOynYRf.png)

```markdown
程序员实现BeanPostProcessor规定接口中的方法：

参数一：Spring工厂创建好的对象
参数二：对象名字
返回值：返回的对象会交给Spring框架
Object postProcessBeforeInitiallization(Object bean String beanName)
作用：Spring创建完对象，并进行注入后，会运行Before方法进行加工

Object postProcessAfterInitiallization(Object bean String beanName)
作用：Spring执行完对象的初始化操作后，会运行After方法进行加工

实战中：
很少处理Spring的初始化操作：没有必要区分Before After。只需要实现其中的一个After方法即可
注意：
    postProcessBeforeInitiallization(){
    	return bean对象
    }
```

#### 3. BeanPostProcessor的开发步骤

##### 1. 实现 BeanPostProcessor接口

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

        Categroy categroy = (Categroy) bean;
        categroy.setName("xiaowb");


        return categroy;
    }
}
```

##### 2. Spring的配置文件中进行配置

~~~xml
<bean id="myBeanPostProcessor" class="xxx.MyBeanPostProcessor"/>
~~~

##### 3. BeanPostProcessor细节

==BeanPostProcessor会对Spring工厂中所有创建的对象进行加工。==