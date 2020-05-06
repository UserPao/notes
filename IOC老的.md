## Spring 作者 Rod Johnson 设计了两个接口用以表示容器。

- BeanFactory

- ApplicationContext

  ### BeanFactory

  - 提供IOC的配置机制
  - 包含Bean的各种定义，便于实例化Bean
  - 建立Bean之间的依赖关系
  - Bean生命周期的控制

  ### ApplicationContext

  - BeanFactory是Spring框架的基础设施，面向Spring
  - AppliacitonContext面向使用spring框架的开发者
  - 功能
    - BeanFactory：能够管理，装配Bean
    - ResourcePatternResolver：能够加载资源文件
    - MessageSource：能够实现国际化等功能
    - ApplicationEventPublisher：能够注册监听器，实现监听机制

解释了低级容器和高级容器，我们可以看看一个 IoC 启动过程是什么样子的。说白了，就是 ClassPathXmlApplicationContext 这个类，在启动时，都做了啥。（由于我这是 interface21 的代码，肯定和你的 Spring 4.x 系列不同）。

这里再用文字来描述这个过程：

1. 用户构造 ClassPathXmlApplicationContext（简称 CPAC）
2. CPAC 首先访问了 “抽象高级容器” 的 final 的 refresh 方法，这个方法是模板方法。所以要回调子类（低级容器）的 refreshBeanFactory 方法，这个方法的作用是使用低级容器加载所有 BeanDefinition 和  Properties 到容器中。
3. 低级容器加载成功后，高级容器开始处理一些回调，例如 Bean 后置处理器。回调 setBeanFactory 方法。或者注册监听器等，发布事件，实例化单例 Bean 等等功能，这些功能，随着 Spring 的不断升级，功能越来越多，很多人在这里迷失了方向 ：）。

简单说就是：

1. 低级容器 加载配置文件（从 XML，数据库，Applet），并解析成 BeanDefinition 到低级容器中。

2. 加载成功后，高级容器启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。

   

好，当我们创建好容器，就会使用 getBean 方法，获取 Bean，而 getBean 的流程如下：

![img](https://upload-images.jianshu.io/upload_images/4236553-da9a2f92e4dfa9db.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

从图中可以看出，getBean 的操作都是在低级容器里操作的。其中有个递归操作，这个是什么意思呢？

假设 ： 当 Bean_A 依赖着 Bean_B，而这个 Bean_A 在加载的时候，其配置的 ref = “Bean_B” 在解析的时候只是一个占位符，被放入了 Bean_A 的属性集合中，当调用 getBean 时，需要真正 Bean_B 注入到 Bean_A 内部时，就需要从容器中获取这个 Bean_B，因此产生了递归。

为什么不是在加载的时候，就直接注入呢？因为加载的顺序不同，很可能 Bean_A 依赖的 Bean_B 还没有加载好，也就无法从容器中获取，你不能要求用户把 Bean 的加载顺序排列好，这是不人道的。想66

所以，Spring 将其分为了 2 个步骤：

1. 加载所有的 Bean 配置成 BeanDefinition 到容器中，如果 Bean 有依赖关系，则使用占位符暂时代替。
2. 然后，在调用 getBean 的时候，进行真正的依赖注入，即如果碰到了属性是 ref 的（占位符），那么就从容器里获取这个 Bean，然后注入到实例中 —— 称之为依赖注入。

可以看到，依赖注入实际上，只需要 “低级容器” 就可以实现。

所以 ApplicationContext refresh 方法里面的操作不只是 IoC，是高级容器的所有功能（包括 IoC），IoC 的功能在低级容器里就可以实现。

### getBean的代码逻辑

- 转换beanName
- 从缓存中加载实例
- 实例化Bean
- 检测parentBeanFactory
- 初始化依赖的Bean
- 创建Bean

说了这么多，不知道你有没有理解Spring  IoC？ 这里小结一下：IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

a. 加载配置文件，解析成 BeanDefinition 放在 Map 里。

b. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出  Class 对象进行实例化，同时，如果有依赖关系，将递归调用  getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

可以预见，随着 Spring 的不断发展，高级容器的功能会越来越多。

诚然，了解 IoC 的过程，实际上为了了解 Spring 初始化时，各个接口的回调时机。例如 InitializingBean，BeanFactoryAware，ApplicationListener 等等接口，这些接口的作用，笔者之前写过一篇文章进行介绍，有兴趣可以看一下，关键字：[Spring 必知必会 扩展接口](https://www.google.com/search?q=Spring+必知必会+扩展接口&oq=Spring+必知必会+扩展接口&aqs=chrome..69i57j69i65.626j0j1&sourceid=chrome&ie=UTF-8)。

但是请注意，实现 Spring 接口代表着你这个应用就绑定死 Spring 了！代表 Spring 具有侵入性！要知道，Spring 发布时，无侵入性就是他最大的宣传点之一 —— 即 IoC 容器可以随便更换，代码无需变动。而现如今，Spring 已然成为 J2EE 社区准官方解决方案，也没有了所谓的侵入性这个说法。因为他就是标准，和 Servlet 一样，你能不实现 Servlet 的接口吗？: -)

## spring ioc的启动过程

https://www.cnblogs.com/firepation/p/9584764.html

### 


