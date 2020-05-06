# 广义理解

**控制反转IOC也可以叫依赖注入DI，因为依赖查找DL现在很少用**，有侵入，IOC利用Java反射机制，AOP利用代理模式，IOC概念看似很抽象，但是很容易理解。说简单点就是将对象交给容器管理，你只需要在spring配置文件中配置对应的bean以及配置相应的属性，让spring容器来生成类的实例对象以及管理对象。在sping容器启动的时候，spring会把你在配置文件中配置的bean都初始化好，然后在你需要调用的时候，就把它已经初始化好的那些bean分配给你需要调用这些bean的类

## IOC带来的好处

 最大限度地降低对象之间的耦合度

有利于**不同组的协同合作和单元测试**

**容器可以自动对你的代码进行初始化**



# 具体实现

## Spring中的两种IOC的容器

**BeanFactory和ApplicationContext。**

1. BeanFactory

   基础类型IOC容器，，提供完整的IOC支持，默认采用延迟初始化策略（lazy-load），单例会自动加载。只有当客户端对象需要访问某个受管对象的时候，才对受管对象进行初始化以及依赖注入操作。像对配置文件的属性值填充的高级功能，BeanFactory是无法处理的

   **BeanFactory是无法对${}引用资源文件变量进行解析处理**

   - 提供IOC的配置机制
   - 包含Bean的各种定义，便于实例化Bean
   - 建立Bean之间的依赖关系
   - Bean生命周期的控制

2. ApplicaitonContext

   在BeanFactory的基础上构建，除了拥有BeanFactory的所有支持，还提供如事件发布、国际化信息支持等其他高级特性。在该类型容器启动之后，默认全部初始化并绑定完成。
   
3. **两者的区别**

   *BeanFactory**采用了工厂设计模式，负责读取bean配置文档，管理**bean**的加载，实例化，维护**bean**之间的依赖关系，负责**bean**的生命周期。而**ApplicationContext**除了提供上述**BeanFactory**所能提供的功能之外，还提供了更完整的框架功能：国际化支持、**aop**、事务等。同时**BeanFactory**在解析配置文件时并不会初始化对象**,**只有在使用对象**getBean()**才会对该对象进行初始化，而**A**pplicationContext**在解析配置文件时对配置文件中的所有对象都初始化了**,getBean()**方法只是获取对象的过程。*

简单说就是：

1. 低级容器BeanFactory 加载配置文件（从 XML，数据库，Applet），并解析成 BeanDefinition 到低级容器中。
2. 加载成功后，高级容器ApplicaitonContext启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。

## Bean

Bean是被实例化的，组装的以及被Spring容器管理的java对象。本质是上对象

Spring 容器会自动完成@bean对象的实例化

创建应用对象之间的协作关系的行为称为装配（wiring）,这就是依赖注入的本质

### Bean的三种配置方式

#### 1. 传统的XML配置方式

~~~xml
  <bean id="beanFactroy" class="com.stonegeek.service.impl.BeanFactroyImpl" />
~~~

#### 2. 基于注解的方式

​		如果一个类使用了以下注解，那么此类将自动注册成一个bean，不需要再在applicationContext.xml文件定义bean了

- **@Configuration** 将一个类定义为Bean的配置类
- **@Componet("userDao")**  通过Repository定义一个DAO的bean
- **@Repository** 用户对DAO实现类进行注解
- **@Service** 用户对Service实现类进行注解
- **@Controller** 用户对Controller实现类进行注解
- **@Autowired**  默认按类型匹配注入Bean，自动注入，默认情况下required为ture，要求一顶耀找到匹配的Bean，否则报NoSuchBeanDefinitionException
- **@Autowired(required=false)**  容器中没有一个标注变量类型匹配的Bean,忽略NoSuchBeanDefinitionException异常
- **@Qualifier("userDao")**  指定注入userDao Bean的名称(如果一个方法拥有多个入参，在默认情况下Spring自动选择匹配入参类型的Bean进行注入。Spring允许对方法入参标注@Qualifier以指定注入Bean的名称)
- **@Resource("userDao")** 按名称匹配注入Bean，要求提供一个Bean名称的属性，如果属性为空，则自动采用标注处的变量名或方法名作为Bean 的名称
- **@Inject** 按类型匹配注入Bean，没有required属性
- **@PostConstruct**  相当于bean的init-method属性的功能 
- **@PreDestroy**   相当于bean的destroy-method属性的功能

~~~java
@Component("userDao")
public class userDao{......}
~~~

#### 3. 基于类的JAVA Config

通过java类定义spring配置元数据，且直接消除xml配置文件

Spring3.0基于java的配置直接支持下面的注解：

- **@Configuration**
- **@Bean**
- **@Value**
- @DependsOn
- @Primary
- @Lazy
- @Import
- @ImportResource

### Spring Bean的五种种作用域

#### 1.  singleTon

在Spring IOC容器中仅存在一个Bean实例，Bean以单例方式存在，是默认值

~~~xml
<!-- xml方式 -->
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
~~~

~~~java
// java注解方式,默认就是这个 不用加
@Scope("singleton")
~~~

#### 2.prototype

每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new XXBean()

Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象

是ioc容器来进行加载和实例化的，但是销毁是

~~~xml
<!-- xml方式 -->
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>
~~~

~~~java
// java注解方式
@Scope("prototype")
~~~

#### 3. request

会为每个Http请求创建一个Bean实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

```xml
<!-- xml方式 -->
<bean id="account" class="com.foo.DefaultAccount" scope="request"/>
```

```java
// java注解方式
@Scope("request")
```

#### 4.session

会为每个session创建一个Bean实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

```xml
<!-- xml方式 -->
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>
```

```java
// java注解方式
@Scope("prototype")
```

#### 5.globalSession

会为每个全局的Http Session创建一个Bean实例，该作用域仅对于portlet有效。该作用域仅在基于web的Spring ApplicationContext情形下有效。

```xml
<!-- xml方式 -->
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>
```

```java
// java注解方式
@Scope("prototype")
```

### Spring 中的单例 bean 的线程安全问题了解吗？

​		单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### Spring Bean【依赖】注入的三种方式

#### 1.构造器注入

~~~java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public DI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC){
    this.dependencyA = dependencyA;
  	this.dependencyB = dependencyB;
   	this.dependencyC = dependencyC;
}
~~~

#### 2.setter方法注入

~~~java
private DependencyA dependencyA;
 	private DependencyB dependencyB;
 	private DependencyC dependencyC;
 
 	@Autowired
 	public void setDependencyA(DependencyA dependencyA) {
 	this.dependencyA = dependencyA;
 	}
 	 
 	@Autowired
 	public void setDependencyB(DependencyB dependencyB) {
      this.dependencyB = dependencyB;
  }
   
  @Autowired
  public void setDependencyC(DependencyC dependencyC) {
      this.dependencyC = dependencyC;
 }
~~~

#### 3.属性注入【filed】

~~~java
 @Autowired
 private DependencyA dependencyA;
 
 @Autowired
 private DependencyB dependencyB;
   
 @Autowired
 private DependencyC dependencyC;
~~~

#### 对比

基于construct的注入，固定注入顺序，不允许我们创建bean对象之间的循环依赖关系，冗余，不好看

基于Setter的注入，只有当对象需要被注入的时候才会帮助我们注入，而不是在初始化的时候就注入， 如果你使用基于constructor注入，CGLIB不能创建一个代理，迫使你使用基于接口的代理或虚拟的无参数构造函数。缺点：不能将对象设置为final

基于属性filed注入。方便快捷，缺点：

不符合javaBean的规范，而且很可能引起空指针

同时也不能将对象设置为final

类和DI容器高度耦合，不能在外部使用

类不通过反射不能被实例化（例如单元测试中），需要用DI容器去实例化他，更像是集成测试

### Spring IOC容器启动的过程

![1587034584584](E:\文档\学习资料\笔记\面经\SpringIOC整理.assets\1587034584584.png)

首先，ioc容器启动的过程分为两部分，第一阶段是**容器的启动过程**，第二阶段是**Bean的实例化过程**，

在spring中，最基础的容器接口是由**BeanFactory**定义的，而且BeanFacctory的实现类采用的`延迟加载`，也就是说，**在容器启动的时候，并不会进行Bean的实例化，而是当需要某个类的实例的时候，才会进行第二阶段的Bean的实例化**，而ApplicationContext则在容器启动的时候就完成了所有初始化。

#### 第一阶段 容器启动阶段

- **介绍BeanDefinitionRegistry**

  ​		BeanFactory 只是一个接口，它需要一个实现类，`DefaultListableBeanFactory` 就是一个比较常用的实现类，它还实现了 `BeanDefinitionRegisitry` 接口，该接口在容器中担任 Bean 注册的角色。

  ​		打个比方，BeanDefinitionRegistry 就像图书馆上的书架，所有的书是放在书架上的。虽然还书借书都是跟图书馆（也就是 BeanFactory，或者说 BookFactory）打交道，但书架才是图书馆存放各类图书的地方。所以，书架对于图书馆来说，就是他的 BookDefinitionRegistry。

  ​		而每一本书都应该有自己唯一的标识，在容器中每个实例也应该有这样的标识，这些标识是由 `BeanDefinition` 存储的。它**负责保存对象的所有必要信息，例如对象的 class 类型、是否是抽象类、构造方法参数以及其他属性**等等，当客户端向 BeanFactory 请求相应对象时，BeanFactory 会通过这些信息返回一个完备可用的对象实例。

- **加载配置文件【元数据】**

  ​        IOC 的理念是 `Don't call us, we will call you`。当一个类中需要另外一个类的实例时，我们并不用手动去创建，IOC 容器会帮我们完成这个任务，但我们需要告诉它哪些类之间存在依赖，例如 A 类中依赖的是哪个类，B 类依赖的类是在哪个包下面，而这些信息我们可以使用注解或者配置文件的方式告诉 IOC 容器。

  ​		读取配置IOC是如何读取配置文件的，IOC容器读取配置文件的接口为`BeanDefinitionReader`,它会**根据配置文件格式的不同给出不同的实现类**，将配置文件的内容读取并映射到`BeanDefinition`中，整个过程可以通过如下代码表示

  ~~~java
  public static void main(String[] args){
      DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
      BeanFactory container = (BeanFactory)bindViaPropertiesFile(beanRegistry);
      FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider");
      newsProvider.getAndPersistNews();
  }
  public static BeanFactory bindViaPropertiesFile(BeanDefinitionRegistry registry){
      BeanDefinitionRegistry beanRegistry = <某个 BeanDefinitionRegistry 实现类，通常为 DefaultListableBeanFactory>;
      BeanDefinitionReader beanDefinitionReader = new BeanDefinitionReaderImpl(beanRegistry);
      // 读取配置文件的核心方法
      beanDefinitionReader.loadBeanDefinitions("配置文件路径");
      return (BeanFactory)registry;
  }
  ~~~

- **解析配置文件**

  配置文件被解析成一个个的Bean定义，注册到BeanFactory中，这里说的Bean是还**没有初始化的**，只是配置信息都提取出来了，注册到BeanFactory中也只是保存的map<BeanName,beanDefinition>

  然后设置BeanFactory的类加载器，添加几个BeanPostProcessor，手动注册几个特殊的Bean。

  然后是`BeanFactoryPostProcessor`通过postProcessBeanFactory 方法去对BeanDefinition做处理【更改属性】，

  然后是`beanpostProcessor`注册，实例化为了以后bean的实例化的时候使用

  ---------------------

  在上面读取配置文件的步骤中，仅仅是将<bean>中的属性读取出来，只是字符串，最终应用程序确是由各种类型的对象实例构成的，需要将这些字符串转换成类信息，转换过程需要定义规则，这些规则是由PropertyEditor定义，由`CustomEditorConfigurer`帮我们传递给Spring容器。经过PropertyEditor定义的规则将字符串转换成对应的类型信息之后并存储在`BeanDefinition`中，交给BeanDefinitionRegistry管理，存储在ConcurrentHashMap中，容器启动就完成了

  -------------------------------------

  ![1587026379172](E:\文档\学习资料\笔记\面经\SpringIOC整理.assets\1587026379172.png)

### Bean的生命周期【Bean的实例化过程】

![img](https://img2018.cnblogs.com/blog/1508298/201910/1508298-20191028160944066-52045838.png)

　    **单例**情况下，在创建起容器时就同时**自动创建**了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都**是同一个对象**。

　    **多例**情况下，在我们创建容器的时候并**没有实例**化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都**不是同一个对象**

1. Bean的实例化

  		容器在内部实现Bean实例化的时候，采用策略模式来决定使用何种方式初始化Bean实例，InstantiationStrategy 定义了实例化策略的接口，SimpleInstantiationStrategy 继承了它，主要通过 **反射** 来实现对象的实例化。CglibSubclassingInstantiation 继承了 SimpleInstantiationStrategy 以反射方式实例化的功能，并且还有 **CGLIB 动态字节码** 生成实例的功能。容器默认采用后者实现。

​			InstantiationStrategy 实例化对象后并没有直接将对象返回，而是用 BeanWrapper 进行包装，方便后续对此实例进行 **属性的设置。**其设置的依据是通过上面所讲的 PropertyEditor 接口，在第一步构造完成对象之后，Spring 会根据对象实例构造一个 BeanWrapperImpl 实例，然后将之前 CustomEditor-Configurer 注册的PropertyEditor 复制一份给 BeanWrapperImpl 实例这样，当 BeanWrapper 转换类型、设置对象属性值时，就不会无从下手了。

 2. BeanPostProcessor

    当对象实例化完成之后，会将对象实例传到 BeanPostProcessor，这个接口的定义如下：

    ```java
    public interface BeanPostProcessor{
        Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
        Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
    }
    ```

    这个接口为我们扩展对象实例的行为提供了极大的便利，其中 postProcessBeforeInitialization 对应的就是上图中 BeanPostProcessor 的前置处理，postProcessAfterInitialization 对应的就是 上图中 BeanPostProcessor 的后置处理。Spring 的 AOP 就是使用 BeanPostProcessor 来为对象生成相应的代理对象。

	3. Bean的初始化

    在对象实例话过程调用 BeanPostProcessor 的前置处理 之后，会接着检测对象是否实现了 InitializingBean 接口，如果是，则会调用该接口的 afterPropertiesSet 方法进一步调整对象实例的状态。但是，如果仅仅为了做一个初始化动作而去实现一个接口这样未免有点小题大做，因此 Spring 有提供了另一种方法，就是在 <bean> 中配置 init-method 属性，指定一个方法做对象初始化前的操作。到了这个步骤，对象实例化也快接近尾声了。

	4. Bean的销毁

    当所有的一切，该设置的设置，该注入的注入，该调用的调用之后，容器将会检查 singleton 类型的 bean 实例，看起是否实现了 DisposableBean 接口，或者查看对应的 bean 定义是否通过 <bean> 的 destroy-method 属性指定了自定义的对象销毁方法。如果是，就会为该实例注册一个用于对象销毁的回调（Callback），以便在这些 singleton 类型的对象实例销毁之前，执行销毁逻辑。至此，Bean 对象的实例化阶段也完成了。

    ------------------------------------------------------

    - Bean 容器找到配置文件中 Spring Bean 的定义。
    - Bean 容器利用 Java Reflection API 创建一个Bean的实例。
    - 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
    - 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
    - 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
    - 如果Bean实现了 `BeanFactoryAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader` 对象的实例。
    - 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
    - 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
    - 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
    - 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
    - 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
    - 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
    - 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

    -----------------------------------

### getBean源码解析

#### 大致过程解析

- **转换beanName**

  原因：1. name可能会以&字符开头，表明调用者想要获取FactoryBean本身，而不是实现类所创建的bean。在BeanFactory中，Factory实现类和其他的bean存储方式是一致的，即<beanName, bean>,beanName 中没有&字符，我们需要将name中的&去掉才能拿到FactoryBean实例；2. 如果name是一个别名，需要转换成具体的实例名

- 从缓存中获取实例

- 如果实例不为null，并且arg为空，调用getObjectForBeanInstance方法，并按照name规则返回响应的bean实例

- 如果上诉条件不成立，则到父容器中查找beanName对有的bean实例，存在则直接返回

  parentBeanFactory.getBean(nameToLookup, args);

- 若父容器不存在，则进行下一步操作—合并BeanDefinition

- 处理depens-on依赖

- 创建并缓存bean

- 调用getObjectForBeanInstance方法，并按name规则返回相应的bean实例

- 按需转换bean类型，并返回转换后的bean实例

####  beanName转换

~~~java
 // 循环处理 & 字符。比如 name = "&&&&&helloService"，最终会被转成 helloService
// 对于BeanFactory 这个bean的特殊处理
transformedBeanName(){
}

 // 循环调用，一样解决多重别名的问题
canonicalName(){
}		
~~~

#### 从缓存中获取bean实例

~~~java
// allowEarlyReference表示是否允许其他bean引用正在创建中的bean，用于处理循环引用
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
}
 *   <bean id="hello" class="xyz.coolblog.service.Hello">
 *       <property name="world" ref="world"/>
 *   </bean>
 *   <bean id="world" class="xyz.coolblog.service.World">
 *       <property name="hello" ref="hello"/>
 *   </bean>
 * 如上所示，hello 依赖 world，world 又依赖于 hello，他们之间形成了循环依赖。Spring 在构建 
 * hello 这个 bean 时，会检测到它依赖于 world，于是先去实例化 world。实例化 world 时，发现 
 * world 依赖 hello。这个时候容器又要去初始化 hello。由于 hello 已经在初始化进程中了，为了让 
 * world 能完成初始化，这里先让 world 引用正在初始化中的 hello。world 初始化完成后，hello 
 * 就可引用到 world 实例，这样 hello 也就能完成初始了
~~~

| 缓存集合              | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| singletonObjects      | 用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用 |
| earlySingletonObjects | 用于存放还在初始化中的 bean，用于解决循环依赖                |
| singletonFactories    | 用于存放 bean 工厂。bean 工厂所产生的 bean 是还未完成初始化的 bean。如代码所示，bean 工厂所生成的对象最终会被缓存到 earlySingletonObjects 中 |

#### 合并父 BeanDefinition 与子 BeanDefinition

Spring 支持配置继承，在标签中可以使用`parent`属性配置父类 bean。这样子类 bean 可以继承父类 bean 的配置信息，同时也可覆盖父类中的配置。

~~~xml
<bean id="hello" class="xyz.coolblog.innerbean.Hello">
    <property name="content" value="hello"/>
</bean>

<bean id="hello-child" parent="hello">
    <property name="content" value="I`m hello-child"/>
</bean>
~~~

大体意思是  ，就算hello-child并没有配置class信息，但是也能被实例化，因为他可以继承父类配置的class属性

#### 从 FactoryBean 中获取 bean 实例

~~~document
getObjectForBeanInstance 及它所调用的方法主要做了如下几件事情：
1. 检测参数 beanInstance 的类型，如果是非 FactoryBean 类型的 bean，直接返回
2. 检测 FactoryBean 实现类是否单例类型，针对单例和非单例类型进行不同处理
3. 对于单例 FactoryBean，先从缓存里获取 FactoryBean 生成的实例
4. 若缓存未命中，则调用 FactoryBean.getObject() 方法生成实例，并放入缓存中
5. 对于非单例的 FactoryBean，每次直接创建新的实例即可，无需缓存
6. 如果 shouldPostProcess = true，不管是单例还是非单例 FactoryBean 生成的实例，都要进行后置处理
~~~

