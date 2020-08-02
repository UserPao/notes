## 广义理解

面向切面编程【Aspect-Orienred Programming】 ，AOP可以说是对OOP的补充和完善，OOP引入封装继承和多态建立一种对象层次结构，用以模拟公共行为的一个集合。实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建"方面"，从而使得编译器可以在编译期间织入有关"方面"的代码，属于静态代理

## AOP的常用功能

1. 权限控制
2. 缓存控制
3. 事务控制
4. 审计日志
5. 性能监控
6. 分布式追踪
7. 异常处理

## AOP的相关概念

1. Join point（连接点）：程序执行期间的某一个点，例如执行方法或处理异常时候的点。**在 Spring AOP 中，连接点总是表示方法的执行**。

2. Advice（通知）：通知是指一个切面在特定的连接点要做的事情。通知分为方法执行前通知，方法执行后通知，环绕通知等。许多 AOP 框架（包括 Spring）都将通知建模为拦截器，在连接点周围维护一系列拦截器（形成拦截器链），对连接点的方法进行增强。

3. Pointcut（切点）:一个匹配连接点（Join point）的谓词表达式。通知（Advice）与切点表达式关联，并在切点匹配的任何连接点（Join point）（例如，执行具有特定名称的方法）上运行。切点是匹配连接点（Join point）的表达式的概念，是AOP的核心，并且 Spring 默认使用 AspectJ 作为切入点表达式语言。

   1. wildCards通配符

      1. *匹配任意数量的字符
      2. +匹配指定的类以及子类
      3. .. 表达了匹配任意数的子包或者参数

   2. Operators

      1. && 
      2. ||
      3. ！

   3. designators指示器

      1. 匹配方法

         1. execution

            ```java
            // ? 表示可以省略
            execution(
                modifier-pattern?  修饰符匹配
                ret-type-pattern   返回值匹配
                declaring-type-pattern?  类路径匹配
                name-pattern(param-pattern)  方法名匹配
                throws-pattern? 异常类型匹配
            )
            ```

      2. 匹配注解

         1. @target

            ```java
            // 匹配标注有AdminOnly的类底下的方法，要求的annotation的RetentionPolicy级别为Runtime
            @PointCut（"@target（com.imooc.demo.security.AdminOnly)")
            public void annoDemo(){}
            ```

         2. args

            ```java
            // 匹配传入的参数类型标注有AdminOnly注解的方法
            @PointCut（"@args（com.imooc.demo.security.AdminOnly)")
            public void annoDemo(){}
            ```

         3. within

            ```java
            // 匹配标注有Beta的类底下的方法，要求的annotation的RetentionPolicy级别为CLASS
            @PointCut（"@within（com.google.common.annotaions.Beta)")
            public void annoWithDemo(){}
            ```

         4. annotation

            ```java
            // 匹配方法标注有AdminOnly的注解的方法，要求的annotation的RetentionPolicy级别为METHOD
            @PointCut（"@annotation（com.imooc.demo.security.AdminOnly)")
            public void annoDemo(){}
            ```

      3. 匹配包/类型

         1. within

            ~~~java
            // 匹配ss类里的所有的方法
            @pointCut（“within(com.imooc.service.ss)
            public void matchType(){}
            
            // 匹配ss包以及子包下所有类的方法
            @pointCut（“within(com.imooc..*)
            public void matchPackType(){}
            ~~~

      4. 匹配对象

         1. this

            ~~~java
            // 匹配aop对象的目标对象为指定类型的方法，即DemoDao的aop代理对象的方法
            @PointCut（“this（com.immoc.DemoDao)")
            public void thisDemo(){}
            ~~~

         2. bean

            ~~~java
            // 匹配所有以Service结尾的bean里的方法
            @PointCut（“bean（*Service)")
            public void beanDemo(){}
            ~~~

         3. target

            ~~~java
            // 匹配实现IDao接口的目标对象（而不是aop代理后的对象）的方法，这里即DemoDao的方法
            @PointCut（“target（com.immoc.IDao)")
            public void targetDemo(){}
            ~~~

      5. 匹配参数

         args

         

4. Aspect（切面）：它是一个跨越多个类的模块化的关注点，它是通知（Advice）和切点（Pointcut）合起来的抽象，它定义了一个切点（Pointcut）用来匹配连接点（Join point），也就是需要对需要拦截的那些方法进行定义；它定义了一系列的通知（Advice）用来对拦截到的方法进行增强；

5. Target object（目标对象）：被一个或者多个切面（Aspect）通知的对象，也就是需要被 AOP 进行拦截对方法进行增强（使用通知）的对象，也称为被通知的对象。由于在 AOP 里面使用运行时代理，所以目标对象一直是被代理的对象。

6. AOP proxy（AOP 代理）：为了实现切面（Aspect）功能使用 AOP 框架创建一个对象，在 Spring 框架里面一个 AOP 代理要么指 JDK 动态代理，要么指 CgLIB 代理。

7. Weaving（织入）：是将切面应用到目标对象的过程，这个过程可以是在编译时（例如使用 AspectJ 编译器），类加载时，运行时完成。Spring AOP 和其它纯 Java AOP 框架一样，是在运行时执行植入。

8. Advisor：这个概念是从 Spring 1.2的 AOP 支持中提出的，一个 Advisor 相当于一个小型的切面，不同的是它只有一个通知（Advice），Advisor 在事务管理里面会经常遇到，



## Adivice

- **@Before前置通知**
- **@After后置通知**
- **@AfterThrowing异常通知**
- **@AfterReturning返回通知**
- **@Around环绕通知**

## 三种织入方式

- 编译时织入

  在代码编译时，把切面代码融合进来，生成完整功能的java字节码，需要特殊的编译器如AspectJ

- 类加载的时候织入

  在java字节码加载的时候，把切面的字节码融合进来，同样需要特殊的编译器如AspectJ和AspectWerkz

- 运行时织入

  运行时，通过动态代理的方式调用切面代码，增强业务功能，spring采用这种方式，动态代理有性能开销，但好处明显，不需要特殊编译器和类加载器

## 两种方式生成代理对象

具体使用那种生成由AopProxyFactory根据AdvisedSupport对象的配置来决定

默认策略是如果目标类是接口，则使用JDKProxy，否则使用cglib

- jdkProxy

  使用反射来接收被代理的类，通过Proxy类动态生成代理类，并且要求类实现接口InvocationHandler

  **只能针对有接口的类的接口方法进行动态代理**

  **不能对private进行代理**

- Cglib

  以继承的方式，通过修改字节码生成目标的动态代理

  **某个类被标记为final或者static，无法使用cglib**

  **无法对private，static方法进行代理**

  借助ASM实现，ASM是一个能够操作字节码的框架

- 对比

  反射机制在生层类的过程中比较高效

  ASM在生成类之后的执行过程中比较高效

## spring中如何选择哪种代理方式

- 如果目标对象实现了接口，默认使用JDK动态代理

- 如果目标对象没有实现接口，使用Cglib进行动态代理

- 如果目标对象实现了接口，且强制cglib代理。则使用cglib代理

  ```java
  // 强制使用Cglib代理
  @EnableAspectJAutoProxy(proxyTargetClass = true)
  ```

## 多个AOP的链式调用

![1584881956274](\AOP.assets\1584881956274.png)



## 代理模式

接口 + 真实实现类 + 代理类

### spring中代理模式的实现

真正实现类的逻辑包含在了getBean方法中

getBean方法返回的实际上是Proxy的实例

Proxy是Spring采用JDKProxy或者Cglib动态生成的

## AOP的问题

无法实现内部调用

```java
@Component
public class MenuService {
    @Cache(cacheName = {"menu"})
    public List<String> getMenuList() {
        return "";
    }
    // 走了内部调用，缓存注解会不生效，因为相当于调用的this.getMenuList()，但是this并不是代理的Bean
    public List<String> getMenuList2(){
        return getMenuList();
    }
}
```

## AOP和拦截器以及过滤器的区别

**Filter过滤器：**

​		拦截web访问url地址。

​		过滤器是在面向切面编程中应用的，就是在你的service或者一个方法前调用一个方法，或者在方法后调用一个方法。是基于JAVA的反射机制。过滤器不是在web.xml，比如struts在struts.xml中配置，

**Interceptor拦截器：**

​		拦截以 .action结尾的url，拦截Action的访问。

​		Servlet中的过滤器Filter是实现了统一设置编码，简化操作；同时还可进行逻辑判断，如用户是否已经登陆、有没有权限访问该页面等等工作。它是随你的web应用启动而启动的，只初始化一次，以后就可以拦截相关请求，只有当你的web应用停止或重新部署的时候才销毁

**Spring AOP拦截器：**

​		只能拦截Spring管理Bean的访问（业务层Service）

Spring AOP，是AOP的一种实现，使用的是代理模式。

Filter(过滤器)是J2EE的规范，Servlet2.3开始引入/实现的是职责链模式。Filter可以用来设置字符集、控制权限、控制转向等等。Filter也是AOP的一种实现。

Interceptor (拦截器)，是Struct2中的概念。同样是AOP的一种实现。

Filter与Interceptor联系与区别

1. 拦截器是基于java的反射机制，使用代理模式，而过滤器是基于函数回调。 
2. 拦截器不依赖servlet容器，过滤器依赖于servlet容器。
3. 拦截器只能对action起作用，而过滤器可以对几乎所有的请求起作用（可以保护资源）。
4. 拦截器可以访问action上下文，堆栈里面的对象，而过滤器不可以。  
5. 执行顺序：过滤前-拦截钱-Action处理-拦截后-过滤后。

 

 