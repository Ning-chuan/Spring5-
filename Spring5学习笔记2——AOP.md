# Spring5学习笔记2——AOP

[TOC]

## 第二部分：AOP编程

### 第一章、静态代理设计模式

#### 1. 为什么需要代理设计模式

- 在JavaEE分层开发开发中，那个层次对于我们来讲最重要

  ```markdown
  DAO ---> Service --> Controller 
  
  JavaEE分层开发中，最为重要的是Service层
  ```

- Service层中包含了哪些代码？

  ```markdown
  Service层中 = 核心功能(几十行 上百代码) + 额外功能(附加功能)
  1. 核心功能
     业务运算
     DAO调用
  2. 额外功能(事务、日志、性能...)
     1. 不属于业务
     2. 可有可无
     3. 代码量很小 
  ```

- 额外功能书写在Service层中好不好？

  Controller层（Service层的调用者）除了需要核心功能，还需要这些额外功能。

  但是从==软件设计者角度看：Service层最好不要写额外功能。==

- 现实生活中的解决方式

  ![image-20200422110206172](https://i.loli.net/2020/10/06/A1MTryvRL7sQ6mj.png)

#### 2. 代理设计模式分析

##### 2.1 概念

​	==通过代理类，为原始类（目标）增加额外的功能==
​	好处：利于原始类(目标)的维护

##### 2.2 名词解释

```markdown
1. 目标类 原始类 
   指的是 业务类 (核心功能 --> 业务运算 DAO调用)
2. 目标方法，原始方法
   目标类(原始类)中的方法 就是目标方法(原始方法)
3. 额外功能 (附加功能)
   日志，事务，性能
```

##### 2.3 代理开发的核心要素

```markdown
代理类 = 实现和目标类相同的接口 + 在同名方法中添加额外功能 + 调用原始类同名方法

房东 ---> public interface UserService{
               m1
               m2
          }
          UserServiceImpl implements UserService{
               m1 ---> 业务运算 DAO调用
               m2 
          }
          UserServiceProxy implements UserService
               m1
               m2
```

##### 2.4 编码

**静态代理**：为每一个原始类，手动编写一个代理类 (.java .class)

![image-20200422114654195](https://i.loli.net/2020/10/06/I5rW9jTDznO4hpm.png)

##### 2.5 静态代理存在的问题

```markdown
1. 静态类文件数量过多，不利于项目管理
   UserServiceImpl  UserServiceProxy
   OrderServiceImpl OrderServiceProxy
2. 额外功能维护性差
   代理类中 额外功能修改复杂(麻烦)
```

### 第二章、Spring的动态代理开发

#### 1.  Spring动态代理的概念

```markdwon
概念：通过代理类为原始类(目标类)增加额外功能,代理类由Spring动态生成。
好处：利于原始类(目标类)的维护
```

#### 2. 搭建开发环境

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aop</artifactId>
  <version>5.1.14.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjrt</artifactId>
  <version>1.8.8</version>
</dependency>

<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.8.3</version>
</dependency>
```

#### 3. Spring动态代理的开发步骤

##### 3.1 创建原始对象(目标对象)

```java
public class UserServiceImpl implements UserService{
    @Override
    public void register(User user) {
        System.out.println("UserServiceImpl.register");
    }

    @Override
    public boolean login(String name, String password) {
        System.out.println("UserServiceImpl.login");
        return true;
    }
}
```

```xml
<bean id="userServiceImpl" class="com.yuziyan.factory.UserServiceImpl"></bean>
```

##### 3.2 定义额外功能（实现MethodBeforeAdvice接口）

```java
public class Before implements MethodBeforeAdvice {
	//作用：给原始方法添加额外功能
    //注意：会在原始方法运行之前运行此方法
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("-----method before advice log------");
    }
}
```

```xml
<bean id="before" class="com.yuziyan.dynamic.Before"/>
```

##### 3.3 定义切入点

```markdown
1. 切入点：额外功能加入的位置

2. 目的：由程序员根据自己的需要，决定额外功能加入给那个原始方法
register()
login()

简单的测试：所有方法都做为切入点，都加入额外的功能。
```

```xml
<aop:config>
   <aop:pointcut id="pc" expression="execution(* *(..))"/>
</aop:config>
```

##### 3.4 组装

```xml
<!-- 组装切入点与额外功能 -->
<aop:advisor advice-ref="before" pointcut-ref="pc"/>
```

##### 3.5 调用

```markdown
目的：获得Spring工厂创建的动态代理对象，并进行调用
ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
注意：
   1. Spring的工厂通过原始对象的id值获得的是代理对象
   2. 获得代理对象后，可以通过声明接口类型，进行对象的存储
   
UserService userService=(UserService)ctx.getBean("userService");

userService.login("")
userService.register()
```

#### 4. 动态代理细节分析

##### 4.1 Spring创建的动态代理类在哪里？

```markdown
Spring框架在运行时，通过动态字节码技术，在JVM创建的，运行在JVM内部，等程序结束后就消失了。

什么叫动态字节码技术:通过第三方动态字节码框架，在JVM中创建对应类的字节码，进而创建对象，当虚拟机结束，动态字节码跟着消失。

结论：动态代理不需要定义类文件，都是JVM运行过程中动态创建的，所以不会造成静态代理，类文件数量过多，影响项目管理的问题。
```

![image-20200423165547079](https://i.loli.net/2020/10/06/IjoFcfXKdqh2WbE.png)

##### 4.2 动态代理编程简化代理的开发

在额外功能不改变的前提下，创建其他目标类（原始类）的代理对象时，只需要指定原始(目标)对象即可。

##### 4.3 动态代理额外功能的维护性大大增强

### 第三章、Spring动态代理详解

#### 1. 额外功能的详解

- MethodBeforeAdvice分析

  作用：原始方法执行之前，运行额外功能。

  ```java
  public class Before implements MethodBeforeAdvice {
      /**
       * 作用：给原始方法添加额外功能
       * 注意：会在原始方法运行之前运行此方法
       * @param method 原始方法 login() register() ...
       * @param objects 原始方法的参数列表 name password ...
       * @param o 原始对象 UserServiceImpl OrderServiceImpl
       * @throws Throwable 抛出的异常
       */
      @Override
      public void before(Method method, Object[] objects, Object o) throws Throwable {
          System.out.println("---- MethodBeforeAdvice  log... ----");
      }
  }
  ```

  实战：需要时才用，可能都会用到，也有可能都不用。

- MethodInterceptor(方法拦截器)

  `MethodInterceptor`接口：额外功能可定义在原始方法执行 前、后、前和后。

  ```java
  public class Around implements MethodInterceptor {
      /**
       * @param invocation 封装了原始方法 invocation.proceed()表示原始方法的运行
       * @return 原始方法的返回值
       * @throws Throwable 可能抛出的异常
       */
      @Override
      public Object invoke(MethodInvocation invocation) throws Throwable {
          System.out.println("------ 额外功能 log -----");
  		//原始方法的执行
          Object ret = invocation.proceed();
          //返回原始方法的返回值
          return ret;
      }
  }
  ```

  额外功能运行在原始方法执行之后

  ```java
  public Object invoke(MethodInvocation invocation) throws Throwable {
    Object ret = invocation.proceed();
    System.out.println("-----额外功能运行在原始方法执行之后----");
  
    return ret;
  }
  ```

  额外功能运行在原始方法执行之前和之后（实战：事务需要在之前和之后都运行）

  ```java
  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable {
    System.out.println("-----额外功能运行在原始方法执行之前----");
    Object ret = invocation.proceed();
    System.out.println("-----额外功能运行在原始方法执行之后----");
  
    return ret;
  }
  ```

  额外功能运行在原始方法抛出异常时

  ```java
  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable {
  
    Object ret = null;
    try {
      ret = invocation.proceed();
    } catch (Throwable throwable) {
      System.out.println("-----原始方法抛出异常 执行的额外功能 ---- ");
      throwable.printStackTrace();
    }
  
    return ret;
  }
  ```

  MethodInterceptor可以影响原始方法的返回值

  ```java
  @Override
  public Object invoke(MethodInvocation invocation) throws Throwable {
     System.out.println("------log-----");
     Object ret = invocation.proceed();
     //拿到原始方法的返回值后进行一些操作就会影响，直接返回就不影响
     return false;
  }
  ```

#### 2. 切入点详解

==切入点决定了额外功能加入的位置。==

```xml
<aop:pointcut id="pc" expression="execution(* *(..))"/>
exection(* *(..)) ---> 匹配了所有方法    a  b  c 

1. execution()  切入点函数
2. * *(..)      切入点表达式 
```

##### 2.1 切入点表达式

- 方法切入点表达式：

  ​	![image-20200425105040237](https://i.loli.net/2020/10/06/sODzSLwYu75tvTc.png)

  ```markdown
  *  *(..)  --> 所有方法
  
  * ---> 修饰符 返回值
  * ---> 方法名
  ()---> 参数表
  ..---> 对于参数没有要求 (0个或多个)
  ```

  举例：

  ```markdown
  # 定义login方法作为切入点
  * login(..)
  
  # 定义register作为切入点
  * register(..)
  
  # 定义名字为login且有两个字符串类型参数的方法 作为切入点
  * login(String,String)
  
  # 注意：非java.lang包中的类型，必须要写全限定名
  * register(com.yuziyan.proxy.User)
  
  # ..可以和具体的参数类型连用(至少有一个参数是String类型)
  * login(String,..)
  
  # 精准方法切入点限定
  # 修饰符 返回值    包.类.方法(参数)
  
      *             com.yuziayn.proxy.UserServiceImpl.login(..)
      *             com.yuziyan.proxy.UserServiceImpl.login(String,String)
  ```

- 类切入点表达式：

  ==指定特定的类作为切入点，即这个类中所有的方法都会加上额外功能。==

  举例：

  ```markdown
  # 类中的所有方法都加入额外功能 
  * com.yuziyan.proxy.UserServiceImpl.*(..)
  
  # 忽略包
  # 1. 类只在一级包下  com.UserServiceImpl
  * *.UserServiceImpl.*(..)
  
  # 2. 类可在多级包下  com.yuziyan.proxy.UserServiceImpl
  * *..UserServiceImpl.*(..)
  ```

- 包切入点表达式：

  ==指定包作为切入点，即这个包中的所有类及其方法都会加入额外的功能。==

  举例：

  ```markdown
  # proxy包作为切入点，即proxy包下所有类中的所有方法都会加入额外功能，但是不包括其子包中的类！
  * com.yuziyan.proxy.*.*(..)
  
  # 当前包及其子包都生效
  * com.yuziyan.proxy..*.*(..) 
  ```

##### 2.5 切入点函数

==作用：用于执行切入点表达式。==

1. `execution()`

   ```markdown
   最为重要的切入点函数，功能最全！
   用于执行：方法切入点表达式、类切入点表达式、包切入点表达式 
   
   弊端：execution执行切入点表达式 ，书写麻烦
        execution(* com.yuziyan.proxy..*.*(..))
        
   注意：其他的切入点函数 只是简化execution书写复杂度，功能上完全一致
   ```

2. `args()`

   ```markdown
   # 作用：用于函数(方法)参数的匹配
   
   # 举例：方法参数必须得是2个字符串类型的参数
   	execution(* *(String,String))
   	等价于：
   	args(String,String)
   ```

3. `within()`

   ```markdown
   # 作用：用于进行类、包切入点表达式的匹配
   # 举例：
   # UserServiceImpl类作为切入点：
   	execution(* *..UserServiceImpl.*(..))
   
   	within(*..UserServiceImpl)
   # proxy包作为切入点：
   	execution(* com.yuziyan.proxy..*.*(..))
   
   	within(com.yuziayan.proxy..*)
   ```

4. `@annotation()`

   ```xml
   <!-- 作用：为具有特殊注解的方法加入额外功能 -->
   
   <aop:pointcut id="" expression="@annotation(com.baizhiedu.Log)"/>
   ```

5. 切入点函数间的逻辑运算：

   目的：整合多个切入点函数一起配合工作，进而完成更为复杂的需求。

   - and 与操作（同时满足）

     ```markdown
     # 案例：方法名为login，同时有2个字符串类型的参数：
     	execution(* login(String,String))
     
     	execution(* login(..)) and args(String,String)
     
     # 注意：与操作不能用于同种类型的切入点函数 
     # 错误案例：register方法 和 login方法作为切入点（不能用and，而用or！）
     	execution(* login(..)) and execution(* register(..))
     # 上面的语句会发生错误，因为其实际表达的含义是方法名为login同时方法名为register，显然有悖逻辑，此时应该用到的是 or
     ```

   - or 或操作（满足一种即可）

     ```markdown
     # 案例：register方法 和 login方法作为切入点 
     	execution(* login(..)) or  execution(* register(..))
     ```

### 第四章、AOP编程

#### 1.  AOP概念

```markdown
AOP (Aspect Oriented Programing)   面向切面编程 = Spring动态代理开发
以切面为基本单位的程序开发，通过切面间的彼此协同，相互调用，完成程序的构建
切面 = 切入点 + 额外功能

OOP (Object Oriented Programing)   面向对象编程 Java
以对象为基本单位的程序开发，通过对象间的彼此协同，相互调用，完成程序的构建

POP (Procedure Oriented Programing) 面向过程(方法、函数)编程 C 
以过程为基本单位的程序开发，通过过程间的彼此协同，相互调用，完成程序的构建
```

```markdown
AOP的概念：
     本质就是Spring的动态代理开发，通过代理类为原始类增加额外功能。
     好处：利于原始类的维护

注意：AOP编程不可能取代OOP，而是OOP编程的补充。
```

#### 2. AOP编程的开发步骤

1. 原始对象
2. 额外功能 (MethodInterceptor)
3. 切入点
4. 组装切面 (额外功能+切入点)

#### 3. 切面的名词解释

```markdown
切面 = 切入点 + 额外功能 

几何学
   面 = 点 + 相同的性质
```

<img src="https://i.loli.net/2020/10/06/UzpYE6h83I5jbXN.png" alt="image-20200427134740273" style="zoom: 67%;" />

### 第五章、AOP的底层实现原理

#### 1. 核心问题

- AOP如何创建动态代理类？(动态字节码技术)

- Spring工厂如何加工创建代理对象？通过原始对象的id值，获得的是代理对象。

#### 2. 动态代理类的创建

##### 2.1  JDK的动态代理

- `Proxy.newProxyInstance()`方法参数详解:

  ![image-20200428175248912](https://i.loli.net/2020/10/06/WexSpzTaAZVds7D.png)

![image-20200428175316276](https://i.loli.net/2020/10/06/dQeigqUTlVLufxs.png)

- 编码：

  ```java
  public class TestJDKProxy {
      public static void main(String[] args) {
          //1.创建原始对象
          //注意：由于后面匿名子类的方法中用到了userService，所以应该用final修饰
          //     而JDK1.8以后默认加了final，不需要手动加
          UserService userService = new UserServiceImpl();
  
          //2.JDK创建代理对象
          InvocationHandler handler = new InvocationHandler() {
              @Override
              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                  System.out.println("----------- JDKProxy log -----------");\
                  //目标方法运行：
                  Object ret = method.invoke(userService, args);
                  return ret;
              }
          };
  
          UserService userServiceProxy = (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(), userService.getClass().getInterfaces(),handler);
  
          userServiceProxy.login("海绵宝宝", "1111");
          userServiceProxy.register(new User());
  
      }
  }
  ```

##### 2.2 CGlib的动态代理

- 原理：通过父子继承关系创建代理对象。原始类作为父类，代理类作为子类，这样既可以保证2者方法一致，同时在代理类中提供新的实现(额外功能+原始方法)

![image-20200429111709226](https://i.loli.net/2020/10/06/strbj2U34IkvXCZ.png)

- CGlib编码：

  ```java
  package com.yuziyan.cglib;
  
  import org.springframework.cglib.proxy.Enhancer;
  import org.springframework.cglib.proxy.MethodInterceptor;
  import org.springframework.cglib.proxy.MethodProxy;
  
  import java.lang.reflect.Method;
  
  public class TestCGlibProxy {
      public static void main(String[] args) {
          //1.创建原始对象
          UserServiceImpl userService = new UserServiceImpl();
  
          //2.通过CGlib创建代理对象
          //  2.1 创建Enhancer
          Enhancer enhancer = new Enhancer();
          //  2.2 设置借用类加载器
          enhancer.setClassLoader(TestCGlibProxy.class.getClassLoader());
          //  2.3 设置父类（目标类）
          enhancer.setSuperclass(userService.getClass());
          //  2.4 设置回调，额外功能写在里面
          enhancer.setCallback(new MethodInterceptor() {
              //相当于 InvocationHandler --> invoke()
              @Override
              public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                  //额外功能：
                  System.out.println("========= CGlibProxy log ========");
                  //目标方法执行：
                  Object ret = method.invoke(userService, objects);
                  return ret;
              }
          });
          //  2.5 通过Enhancer对象创建代理
          UserServiceImpl service = (UserServiceImpl) enhancer.create();
  
          //测试：
          service.register();
          service.login();
  
      }
  }
  ```

##### 2.3 总结

```markdown
1. JDK动态代理   Proxy.newProxyInstance()  通过目标类实现的接口创建代理类 
2. Cglib动态代理 Enhancer                  通过继承目标类创建代理类 
```

#### 3. Spring工厂如何返回代理对象

- 思路分析：

![image-20200430113353205](https://i.loli.net/2020/10/06/8RAKqTWgHh75XFs.png)

- 编码模拟：

  ```java
  public class ProxyBeanPostProcessor implements BeanPostProcessor {
      @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          return bean;
      }
  
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          InvocationHandler invocation = new InvocationHandler() {
              @Override
              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                  System.out.println("----------- 模拟Spring返回代理对象的方式 log -----------");
  
                  Object ret = method.invoke(bean, args);
  
                  return ret;
              }
          };
  
          return Proxy.newProxyInstance(ProxyBeanPostProcessor.class.getClassLoader(), bean.getClass().getInterfaces(), invocation);
      }
  }
  ```

  ```xml
  <!-- 1.配置原始对象 -->
  <bean id="userService" class="com.yuziyan.factory.UserServiceImpl"></bean>
  <!-- 2.配置自己模拟的ProxyBeanPostProcessor -->
  <bean id="proxyBeanPostProcessor" class="com.yuziyan.factory.ProxyBeanPostProcessor"/>
  ```

### 第六章、基于注解的AOP编程

#### 1. 开发步骤：

1. 原始对象

2. 额外功能

3. 切入点

4. 组装切面

   ```java
   /**
    * 声明切面类     @Aspect
    * 定义额外功能   @Around
    * 定义切入点     @Around("execution(* login(..))")
    *
    */
   @Aspect
   public class MyAspect {
   
       @Around("execution(* login(..))")//组装了切入点和额外功能
       public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
           //额外功能：
           System.out.println("--------- 基于注解的AOP编程 log --------");
           //原始方法执行：
           Object ret = joinPoint.proceed();
   
           return ret;
       }
   }
   ```

   ```xml
   <!-- 原始对象 -->
   <bean id="userService" class="com.yuziyan.aspect.UserServiceImpl"></bean>
   
   <!-- 切面 -->
   <bean id="myAspect" class="com.yuziyan.aspect.MyAspect"/>
   
   <!-- 开启基于注解的AOP编程 -->
   <aop:aspectj-autoproxy/>
   ```

#### 2. 细节分析：

- 切入点复用：

  ```java
  @Aspect
  public class MyAspect {
  
      /**
       * 切入点复用：定义一个函数，加上@Pointcut注解，通过该注解的value定义切入点表达式，以后可以复用。
       */
      @Pointcut("execution(* login(..))")
      public void myPointcut(){}
  
      @Around("myPointcut()")//组装了切入点和额外功能
      public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
          //额外功能：
          System.out.println("--------- 基于注解的AOP编程 log --------");
          //原始方法执行：
          Object ret = joinPoint.proceed();
  
          return ret;
      }
  
  
      @Around("myPointcut()")
      public Object around1(ProceedingJoinPoint joinPoint) throws Throwable {
          //额外功能：
          System.out.println("--------- 基于注解的AOP编程 tx --------");
          //原始方法执行：
          Object ret = joinPoint.proceed();
  
          return ret;
      }
  
  }
  ```

- 动态代理的创建方式：

  ```markdown
  AOP底层实现  2种代理创建方式
    1.  JDK   通过实现接口，创建代理对象
    2.  Cglib 通过继承目标类，创建代理对象
    
    默认情况 AOP编程 底层应用JDK动态代理创建方式 
    如果要切换Cglib
         1. 基于注解AOP开发
            <aop:aspectj-autoproxy proxy-target-class="true" />
         2. 传统的AOP开发
            <aop:config proxy-target-class="true">
            </aop>
  ```

### 第七章、AOP开发中的一个坑

坑：在同一个业务类中，业务方法间相互调用时，只有最外层的方法,加入了额外功能(内部的方法，通过普通的方式调用，运行的都是原始方法)。如果想让内层的方法也调用代理对象的方法，就要实现AppicationContextAware获得工厂，进而获得代理对象。

```java
public class UserServiceImpl implements UserService, ApplicationContextAware {
    private ApplicationContext ctx;


    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
              this.ctx = applicationContext;
    }

    @Log
    @Override
    public void register(User user) {
        System.out.println("UserServiceImpl.register 业务运算 + DAO ");
        //throw new RuntimeException("测试异常");

        //调用的是原始对象的login方法 ---> 核心功能
        /*
            设计目的：代理对象的login方法 --->  额外功能+核心功能
            ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext2.xml");
            UserService userService = (UserService) ctx.getBean("userService");
            userService.login();

            Spring工厂重量级资源 一个应用中 应该只创建一个工厂
         */

        UserService userService = (UserService) ctx.getBean("userService");
        userService.login("suns", "123456");
    }

    @Override
    public boolean login(String name, String password) {
        System.out.println("UserServiceImpl.login");
        return true;
    }
}
```

### 第八章、AOP阶段知识总结

![image-20200503162625116](https://i.loli.net/2020/10/08/ABa1f2PLwzQsSFO.png)