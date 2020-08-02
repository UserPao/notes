## Spring事务的隔离级别

TransactionDefinition接口中定义了五个表示隔离级别的常量

- TransactionDefinition.ISOLATION_DEFAULT:

  使用后端数据库默认的隔离级别，mysql默认采用REPEATABLE_READ隔离级别，Oracle默认采用READ_COMMITED隔离级别

- TransactionDefinition.ISOLATION_READ_UNCOMMITTED:

  最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读，幻读和不可重复读

- TransactionDefinition.ISOLATION_READ_COMMITED:

  允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读活不可重复读仍有可能发生

- TransactionDefinition.ISOLATION_REPEATABLE_READ:

  对同一字段的多次读取结果都是一致的，除非数据是被本身事物自己所修改的，可以阻止脏读和不可重复读，但幻读仍然可能发生

- TransactionDefinition.ISOLATION_SERIALIZABLE:

  最高的隔离级别，完全服从ACID的隔离级别，所有的事务一次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读，不可重复读以及幻读，但是这将严重影响程序的性能，通常不设置该级别。

## Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

## Spring事务中的事务传播行为

- 支持当前事务的情况
  - TransactionDefinition.PROPAGATION_REQUIRED:如果当前存在事务，则加入该事务，如果当前没有事务，则创建一个新的事务
  - TransactionDefinition.PROPAGATION_SUPPORTS:如果当前存在事务，则加入该事务，如果当前没有事务，则以非事务的方式继续运行
  - TransactionDefinition.PROPAGATION_MANDATORY:如果当前存在事务，则加入该事务，如果当前没有事务，则抛出异常
- 不支持当前事务的情况
  - TransactionDefinition.PROPAGATION_REQUIRES_NEW:创建一个新的事务，如果当前存在事务，则把当前事务挂起
  - TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起
  - TransactionDefinition.PROPAGATION_NEVER: 以非事务方式运行，如果当前存在事务，则抛出异常
- 其他情况
  - TransactionDefinition.PROPAGATION_NESTED:如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED

## SpringMVC的原理





![1582891545967](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1582891545967.png)

客户端发送请求—> 前端控制器DispatcherServlet接收客户端请求—>找到处理器映射HandlerMapping解析请求对应的Handler—>HandlerApdapter根据Handler来调用真正的处理器处理请求，并处理响应的业务逻辑—>处理器返回模型视图ModelAndView—>视图解析器进行解析—>返回一个视图对象—>前端控制器DispatcherServelet渲染数据—>将得到的视图对象返回给用户

## springBoot和SpringMVC的区别

- 联系：

​         Spring最初利用工厂模式（DI)和代理模式解耦应用组件，为了解耦开发了springmvc；而实际开发过程中，经常会使用到注解，程序的样板很多，于是开发了starter，这套就是springboot。

- 区别：

  -    springboot是约定大于配置，可以简化spring的配置流程；springmvc是基于servlet的mvc框架，个人感觉少了model中的映射。
  - 以前web应用要使用到tomat服务器启动，而springboot内置服务器容器，通过@SpringBootApplication中注解类中main函数启动即可
  - **spring boot只是一个配置工具,整合工具,辅助工具.**
  - **springmvc是框架,项目中实际运行的代码**
  - Spring MVC是基于Servlet 的一个 MVC 框架主要解决 WEB 开发的问题，因为 Spring 的配置非常复杂，各种XML、 JavaConfig、hin处理起来比较繁琐。于是为了简化开发者的使用，从而创造性地推出了Spring boot，约定优于配置，简化了spring的配置流程。

- **Spring 是一个“引擎”；**

  **Spring MVC 是基于Spring的一个 MVC 框架；**

  **Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。**
  
- springBoot如何实现的自动配置

  - conditional注解

## SpringBoot启动过程

### 1.创建SpringApplication对象

```java
initialize(sources);
这个方法做了什么
1.保存主配置类
2.判断当前应用是否是一个web应用
3.从类路径下找到META-INF/spring.factory配置的所有ApplicationContextInitializer，然后保存起来
4.从类路径下找到META-INF/spring.factory配置的所有ApplicationListener
5.找到主程序类，主程序类可以传多个，找带main方法的配置类为主配置类
```

### 2.调用run方法

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
    	// 获取SpringApplicationRunListener，也是从类路径下META-INF/spring.factoies中获取
		SpringApplicationRunListeners listeners = getRunListeners(args);
    	//  将所有监听器启动
		listeners.starting();
		try {
            // 封装命令行参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // 准备环境
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
            // 创建环境完成后，回调所有监听器的envirmentPrepared方法
			configureIgnoreBeanInfo(environment);
            // 打印spring图标
			Banner printedBanner = printBanner(environment);
            // 创建ApplicationContext，决定是创建web环境的ioc容器还是创建普通的ioc容器
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
            // 准备上下文环境，将environment保存到ioc中，而且applyInitializer
            // applyInitializer() 回调之前保存的所有的ApplicationContextInitializer的initialize方法
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            // prepareContext运行完成后回调所有的springApplicaitonRunListener的contextLoad方法
            
            // 刷新容器，ioc容器初始化，如果是web应用还会创建嵌入式的tomcat
			refreshContext(context);
            // 从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
            // ApplicaitonRunner先回调，CommandLineRunner再回调
			afterRefresh(context, applicationArguments);
            // 所有的监听器调用finish方法
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
    	// 整个springboot应用启动后，返回ioc容器
		return context;
	}
```

##  Spring注解的原理

 下面拿spring的controller来当做示例：
         Controller类使用继承@Component注解的方法，将其以单例的形式放入spring容器，如果仔细看的话会发现每个注解里面都有一个默认的value()方法，它的作用是为当前的注解声明一个名字，一般默认为类名，然后spring会通过配置文件中的<context:component-scan>的配置，进行如下操作：

1、使用asm技术扫描.class文件，并将包含@Component及元注解为@Component的注解@Controller、@Service、@Repository或者其他自定义的的bean注册到beanFactory中，

2、然后spring在注册处理器

3、实例化处理器，然后将其放到beanPostFactory中，然后我们就可以在类中进行使用了。

4、创建bean时，会自动调用相应的处理器进行处理。

## 一些常见注解的问题

### @Transactional

1. ### propagation 属性

   需要注意下面三种 propagation 可以不启动事务。本来期望目标方法进行事务管理，但若是错误的配置这三种 propagation，事务将不会发生回滚。

   1. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
   2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
   3. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

2. ### rollbackFor 属性

   默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。

   如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。例：

   @Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)

3. ### @Transactional 只能应用到 public 方法才有效

4. **@Transactional注解只能在抛出RuntimeException或者Error时才会触发事务的回滚**

   常见的非RuntimeException是不会触发事务的回滚的。但是我们平时做业务处理时，需要捕获异常，所以可以手动抛出RuntimeException异常或者添加rollbackFor = Exception.class(也可以指定相应异常)

### @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。

2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。

3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

   `@Bean`注解使用示例：

   ~~~java
   @Configuration
   public class AppConfig {
       @Bean
       public TransferService transferService() {
           return new TransferServiceImpl();
       }
   
   }
   ~~~

   上面的代码相当于下面的xml文件

   ~~~xml
   <beans>
       <bean id="transferService" class="com.acme.TransferServiceImpl"/>
   </beans>
   ~~~

   下面这个例子是通过 `@Component` 无法实现的。

   ~~~java
   @Bean
   public OneService getService(status) {
       case (status)  {
           when 1:
                   return new serviceImpl1();
           when 2:
                   return new serviceImpl2();
           when 3:
                   return new serviceImpl3();
       }
   }
   ~~~

## SpringBoot处理异常的方式

1. **自定义错误异常页面**

   SpringBoot默认的处理异常的机制：SpringBoot默认的已经提供了一套处理异常的机制。一旦程序中出现了异常SpringBoot会像/error的url发送请求。在springBoot中提供了一个叫BasicExceptionController来处理/error请求，然后跳转到默认显示异常的页面来展示异常信息。

   ​    如果我们需要将所有的异常同一跳转到自定义的错误页面，需要再src/main/resources/templates目录下创建error.html页面。注意：名称必须叫error

2. **使用@ExceptionHandler注解处理异常**

   注解使用方式

   ~~~java
   /**
        * java.lang.ArithmeticException
        * 该方法需要返回一个ModelAndView：目的是可以让我们封装异常信息以及视图的指定
        * 参数Exception e:会将产生异常对象注入到方法中
        */
       @ExceptionHandler(value={java.lang.ArithmeticException.class})
       public ModelAndView arithmeticExceptionHandler(Exception e){
           ModelAndView mv = new ModelAndView();
           mv.addObject("error", e.toString());
           mv.setViewName("error1");
           return mv;
       }
   ~~~

3. **@ControllerAdvice+@ExceptionHandler注解处理异常**

   创建一个能够处理异常的全局异常类。在该类上需要添加@ControllerAdvice注解

   ~~~java
   /**
    * 全局异常处理类
    */
   @ControllerAdvice
   public class GlobalException {
       /**
        * java.lang.ArithmeticException
        * 该方法需要返回一个ModelAndView：目的是可以让我们封装异常信息以及视图的指定
        * 参数Exception e:会将产生异常对象注入到方法中
        */
       @ExceptionHandler(value={java.lang.ArithmeticException.class})
       public ModelAndView arithmeticExceptionHandler(Exception e){
           ModelAndView mv = new ModelAndView();
           mv.addObject("error", e.toString());
           mv.setViewName("error1");
           return mv;
       } 
   }
   ~~~

   **@ControllerAdvice的底层实现原理**

   1. 首先,容器启动时，会定义类型为RequestMappingHandlerAdapter的bean组件，这是DispatcherServlet用于执行控制器方法的HandlerAdapter,它实现了接口InitializingBean,所以自身在初始化时其方法#afterPropertiesSet会被调用执行。
   2. 从以上代码可以看出，RequestMappingHandlerAdapter bean组件在自身初始化时调用了#initControllerAdviceCache,从这个方法的名字上就可以看出，这是一个ControllerAdvice相关的初始化函数，而#initControllerAdviceCache具体又做了什么呢
   3. `#initControllerAdviceCache`方法的实现逻辑来看，它将容器中所有使用了注解`@ControllerAdvice`的`bean`或者其方法都分门别类做了统计，记录到了`RequestMappingHandlerAdapter`实例的三个属性中 
      - requestResponseBodyAdvice
        用于记录所有@ControllerAdvice + RequestBodyAdvice/ResponseBodyAdvice bean组件
      - modelAttributeAdviceCache
        用于记录所有 @ControllerAdvice bean组件中的 @ModuleAttribute 方法
      - initBinderAdviceCache
        用于记录所有@ControllerAdvice bean组件中的 @InitBinder 方法
   4. 使用注解`@ControllerAdvice`的`bean`中的信息被提取出来了
   5. 

4. 配置SimpleMappingExceptionResolver处理异常**

   在全局异常类中添加一个方法完成异常的同一处理

   ~~~java
   /**
    * 通过SimpleMappingExceptionResolver做全局异常处理
    */
   @Configuration
   public class GlobalException {
       
       /**
        * 该方法必须要有返回值。返回值类型必须是：SimpleMappingExceptionResolver
        */
       @Bean
       public SimpleMappingExceptionResolver getSimpleMappingExceptionResolver(){
           SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
           
           Properties mappings = new Properties();
           
           /**
            * 参数一：异常的类型，注意必须是异常类型的全名
            * 参数二：视图名称
            */
           mappings.put("java.lang.ArithmeticException", "error1");
           mappings.put("java.lang.NullPointerException","error2");
           
           //设置异常与视图映射信息的
           resolver.setExceptionMappings(mappings);
           
           return resolver;
       }
       
   }
   ~~~

5. **自定义HandlerExceptionResolver类处理异常**

   需要再全局异常处理类中实现HandlerExceptionResolver接口

   ~~~java
   /**
    * 通过实现HandlerExceptionResolver接口做全局异常处理
    */
   @Configuration
   public class GlobalException implements HandlerExceptionResolver {
   
       @Override
       public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
               Exception ex) {
           ModelAndView mv = new ModelAndView();
           //判断不同异常类型，做不同视图跳转
           if(ex instanceof ArithmeticException){
               mv.setViewName("error1");
           }
           
           if(ex instanceof NullPointerException){
               mv.setViewName("error2");
           }
           mv.addObject("error", ex.toString());
           
           return mv;
       }
   }
   ~~~


## SpringBoot自动配置的原理

1. SpringBoot启动的时候加载主配置类（@SpringBootApplication），开启了自动配置功能@EnableAutoConfiguration

2. @EnableAutoConfiguration的作用

   利用AutoConfigurationImportSelector给容器中导入一些组件，可以查看selectImports()方法的内容；

   将类路径下META-INF/spring.factories里面配置的所有AutoConfiguration的值加入到容器中

   ![img](Spring.assets\20180903142348288.png)

   每一个这样的xxxAutoConfiguration类都是容器的一个组件，都加入到容器中，用他们来自动配置

3. 每一个自动配置类进行自动配置功能

4. 以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理；
          一但这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

   ![img](Spring.assets\20180903143011204.png)

     a、@Configuration //表示这是一个配置类，也可以给容器中添加组件
      b、@EnableConfigurationProperties//启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中 。 

    所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；该类中有什么属性，配置文件就可以配置什么；@ConfigurationProperties(prefix = "spring.http.encoding") //从配置文件中获取指定的值和bean的属性进行绑定。
   ![img](Spring.assets\20180903143332472.png)

    c、@ConditionalOnWebApplication //Spring底层@Conditional注解：根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效； 判断当前应用是否是web应用，如果是，当前配置类生效
       d、@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有CharacterEncodingFilter这个类；CharacterEncodingFilter类是SpringMVC中进行乱码解决的过滤器；
          e、@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing=true) //判断配置文件中是否存在 spring.http.encoding.enabled这个配置；如果不存在，判断也是成立的