## JVM的组成

​	jvm主要由**类加载器 Class Loader**，**执行引擎 Exexution Engine**，**运行数据区 Runtime data area**和 **本地接口 Native Interface** 组成

1. 类加载器加载类文件到内存，Class Loader只管加载，只要符合文件结构就加载，至于能不能运行，是由Execution Engine负责。

2. 执行引擎也叫作解释器，负责解释命令，提交操作系统执行

3. 然后去执行

   @ Native Interface： 融合不同开发语言的原生库为Java所用

   @Runtime Data Area：JVM 内存结构模型，程序加载的地方，然后去执行。

## 类加载器的双亲委派机制

JAVA类加载器基于三个机制：**委托，可见性和单一性**

1. 委托机制是指加载一个类的请求交给父类加载器，如果父类加载器不能够找到或者加载这个类，那么在加载他
2. 可见性的原理是指子类的加载器可以看见所有的父类加载器加载的类，而父类加载器看不到子类加载器加载的类
3. 单一性原理是指一个类仅被加载一次，这是由于委派机制确保子类加载器不会再次加载父类加载器加载过的类

双亲委派机制的解析：

可以把每个类加载都想成一个大懒汉，每次让他办事时他都让爸爸代办。没想到爸爸也是个大懒汉，于是爸爸也让他的爸爸代办。这是到了爷爷那里，爷爷也很懒，但是他没有爸爸了，于是只能一边抱怨一边干，然后发现自己做不了，又骂骂咧咧的把活儿交给了自己的儿子，然后爸爸开始干活，发现自己也不能完成这个任务，于是他也是骂骂咧咧的把活交给了儿子，儿子挨了一顿骂，然后开始干活，经过了1小时的苦干，这个活儿终于完事了。

![1578989367158](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1578989367158.png)



1. 原因

   防止多份同样的字节码被多次加载，

   例子：

   在System.out.println()的时候加载System类，假如不是双亲委派机制，会出现多个System的class文件，内存浪费

## 内存模型

### 方法区（线程共享）

​		方法区主要是放一下类似类定义、常量、编译后的代码、静态变量等，在JDK1.7中，HotSpot VM的实现就是将其放在永久代中，这样的好处就是可以直接使用堆中的GC算法来进行管理，但坏处就是经常会出现内存溢出，即PermGen Space异常，所以在JDK1.8中，HotSpot VM取消了永久代，用元空间取而代之，元空间直接使用本地内存，理论上电脑有多少内存它就可以使用多少内存，所以不会再出现PermGen Space异常

#### 元空间（MetaSpace）和永久代（PermGen）的区别

1. 元空间使用本地内存，永久代使用JVM内存
2. 元空间比永久代的优势
   1. 字符串常量池存在永久代中，容易出现性能问题和内存溢出
   2. 类和方法的信息大小很难确定，给永久代的大小指定带来困难
   3. 永久代会为GC带来不必要的复杂性
   4. 方便HotSpot与其他JVM如Jrockit的集成

### 堆（线程共享）

​		几乎所有对象、数组等都是在此分配内存的，在JVM内存中占的比例也是极大的，也是GC垃圾回收的主要阵地。平时我们说的什么新生代、老年代、永久代也是指的这片区域

### 虚拟机栈（线程独占）

​		当JVM在执行方法时，会在此区域中创建一个栈帧来存放方法的各种信息，比如返回值，局部变量表和各种对象引用等，方法开始执行前就先创建栈帧入栈，执行完后就出栈。

​		java方法执行的内存模型

​		包含多个栈帧，栈帧存储局部变量表（方法执行过程中的所有变量），操作数栈（入栈、出栈、复制、交换、产生消费变量），动态链接，返回地址。

​		有固定的容量，会自动释放内存

​		![1578994878466](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1578994878466.png)

​		

### 本地方法栈（线程独占）

​		和虚拟机栈类似，不过区别是专门提供给Native方法用的。

### 程序计数器（线程独占）

​		占用很小的一片区域，我们知道JVM执行代码是一行一行执行字节码，所以需要一个计数器来记录当前执行的字节码行数。

​		如果线程正在执行的是JAVA方法，记录的是正在执行的虚拟机字节码指令的地址

​		如果正在执行的是Native方法，则该值为undefined

​		不会发生内存泄漏

​		是逻辑计数器，而不是物理计数器

### 堆和栈的区别

​		数组，对象实例都保存在堆中，栈定义变量来保存堆中目标的首地址

​		![1579056804162](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579056804162.png)

1. 管理方式

   栈自动释放，堆需要GC

2. 空间大小

   栈比堆小

3. 碎片相关

   栈产生的碎片远小于堆

4. 分配方式

   栈支持静态分配和动态分配，而堆空间仅支持动态分配

5. 效率

   栈比堆要高

### 不同JDK版本的字符串的intern（）方法的区别，JSK6 vs JDK6

~~~java
String s = new String('a")  

s.intern();
~~~

JDK6:当调用intern方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的该字符串的引用。否则，将此字符串对象<b>副本</b>添加到字符串常量池中，并且返回该字符串对象的引用。

JDK6+：当调用intern方法时，如果字符串常量池先前已创建出该字符串对象，则返回池中的概字符串的引用。否则，如果该字符串对象已经存在于java堆中，则将堆中对此对象的引用添加到字符串常量池中，并且返回该引用，如果堆中不存在，则在池中创建该字符串并返回其引用。

~~~java
String s = new String("a");
s.intern();
String s2 = "a";
syso(s == s2);
String s3 = new String("a") + new String("a");
s3.intern();
String s4 = "aa";
syso(s3 == s4);
~~~

1. JDK1.8 

   ~~~java
   true;
   false;
   ~~~

   ![1579070770192](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579070770192.png)

2. JDK1.6

   ~~~java
   false;
   false;
   ~~~

   ![1579070676912](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579070676912.png)

## 反射

​		在运行状态中，对于任意一个类，都能知道这个类的所有属性和方法，对于任意一个对象，都能调用他的任意方法和属性。这种动态获取信息，动态调用对象方法的机制

### 反射的例子

1. 创建一个类

   ~~~java
   public class Robot {
       private String name;
       public void sayHi (String helloSentence){
           System.out.println(helloSentence);
       }
       private String throwHello (String tag) {
           return "hello" + tag;
       }
   }
   ~~~

2.  创建测试类

   ~~~java
   public class ReflectSample{
       public static void main(String [] args) throws ClassNotFindException IllegalAccessException {
           Class rc = Class.forName("com.interView.javaBasic.reflect.Robot");
           Robot r =  (Robot) rc.newInstance();
           syso("class name is :" + rc.getName());
           Method getHello = rc.getDeclaredMethod("throwHello", String.class);
           //  private Method need set true ,else then cannot excute
           getHello.setAccessible(true);
           Object str = getHello.invoke(r, "123");
           syso(str);
           
           //  私有属性
           Field name =  rc.getDeclaredField("name");
           name.setAccessible(true);
           name.set(r, "123");
       }
   }
   ~~~

## 类加载器ClassLoader

### ClassLoader作用

主要工作在Class装载的加载阶段，主要作用是从系统外部获得Class二进制数据流，是java的核心组件，所有的Class都是由ClassLoader进行加载的，ClassLoader负责通过将Class文件里的二进制数据流装载进系统，然后交给JAVA虚拟机进行连接，初始化等操作.

### ClassLoader 的种类

1. **BootStrapClassLoader**

   C++编写，加载核心库java.*

2. **ExtClassLoader:**

   java编写,加载扩展类javax.*

3. **AppClassLoader**

    Java 编写，加载应用类

4. **自定义ClassLoader**

   Java编写，定制化加载

   @@ 自定义ClassLoader的实现

   关键函数

   ~~~java
   protected Class<?> findClass (String name) throws ClassNotFindException {
       throw new ClassNotFoundException(name);
   }
   ~~~

   ~~~java
   protected final Class<?> defineClass(byte[] b	,int off, int len) throws ClassFormatError{
       return defineClass(null, b, off, len, null);
   }
   ~~~

   @@ 示例

   ~~~java
   public class MyClassLoader extends ClassLoader {
       private String path;
       private String classLoaderName;
       public MyClassLoader(String path, String classLoaderName) {
           this.path = path;
           this.classLoaderName = classLoaderName;
       }
       // 用于寻找类文件
       public Class findClass (String name) {
           byte[] b = loadClassData(name);
           return defineClass(name, b, 0, b.length);
       }
       // 用于加载类文件
       private byte[] loadClassData(String name) {
           name = path + name + ".class";
           InputStream in = null;
           ByteArrayOutputStream out = null;
           try{
               in = new FileInputStream(new File(name));
               out = new ByteArrayOutputStream();
               int i = 0;
               while ((i = in.read()) != -1) {
                   out.write(i);
               }
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               out.close();
               in.close();
           }
           retrn out.toByteArray();
       }
   }
   ~~~

## 类的加载方式

1. 隐式加载

   new ClassName()

2. 显示加载

   loadClass、forName()

## loadClass和forName的对比

1. 共同点

   1.都能在运行时，对任意一个类。都能知道该类的属性和方法，对任意一个对象都能调用他的方法和属性

2. 区别

   1. Class.forName得到的class是已经初始化完成的（会执行类的静态代码块）
   2. Classloader.loadClass得到的class是还没有链接的（例如spring的懒加载）

## 类的加载过程

1. 编译

   java文件编译后生成.class字节码文件

2. 加载

   类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例

   -------------------------------------------

   通过classLoader加载.class文件字节码，类加载器会在指定的classpath中找到Student.class（通过类的全限定名）这个文件，然后读取字节流中的数据，将其存储在方法区中。生成class对象，这个对象比较特殊，一般也存放在方法区中，用于作为运行时访问Student类的各种数据的接口。

3. 连接

   1. 校验：检查加载的class的正确性和安全性
   2. 准备：为类变量分配存储空间并设置类变量初始值，并**不会执行赋值的操作**，而是将其初始化为0。
   3. 解析：JVM将常量池内的符号引用转换成直接引用

4. 初始化

   **有父类先初始化父类，然后初始化自己**,执行类变量赋值和类静态代码块

## JVM创建对象的过程

- 【类加载检查-->分配内存-->初始化零值-->设置对象头-->执行init方法】

  - 类装载检查：虚拟机遇到一条new指令时，先检查这个指令的参数能否在常量池中定位到一个类的符号引用，并检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，则先进行类的加载过程。

  - 分配内存：有两种方式

    指针碰撞：假设Java堆中的内存是规整的，用过的内存在一边，空闲的在另一边，中间有一个指针作为分界点的指示器，所分配的内存就把那个指针向空闲那边挪动一段与对象大小相等的距离。

    空闲列表：如果Java堆中的内存不是规整的，虚拟机必须维护一个列表，记录哪些内存块可用的，分配时从列表中找到一块足够大的空间划分给对象，并更新列表的记录。

  - 初始化零值 将分配到的内存空间都初始化为零值，如果用TLAB，则在TLAB分配时初始化为零值。

  - 设置对象头：主要设置类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。

  - 执行init方法初始化。
  
- **变量的初始化顺序**

  **父类静态变量，父类静态代码块，子类静态变量，子类静态代码块，父类非静态变量，父类非静态代码块，父类构造函数，子类非静态变量，子类非静态代码块，子类构造函数。**

## JVM三个性能调优参数 -Xms -Xmx -Xss的含义

~~~shell
java -Xms 128m -Xmx 128m -Xss256k -jar xxx.jar
~~~

-Xss：规定了每个线程虚拟机（堆栈）的大小，控制此进程中并发线程的数量

-Xms：堆的初始大小，该进程该创建出来的时候的专属java堆的大小

-Xmx：堆能达到的最大值，一般和Xms一样，因为当heap不够用时扩容会导致内存抖动，

## java内存模型中堆和栈的区别

### 程序运行时的三种内存分配策略

1. 静态的：编译时就能确定每个数据目标在运行时候的存储空间需求。分配固定，要求程序中不允许有可变数据结构的存在。不允许有嵌套和递归
2. 栈式的：动态的，数据区需求在编译时未知，运行时模块入口前确定
3. 堆式的：动态的。编译时和运行时都无法确定模块运行所需的内存分配，如可变长度串和对象实例

## java对象的访问定位方式

java对象在访问的时候，我们需要通过java虚拟机栈的reference类型的数据去操作具体的对象，由于reference类型在java虚拟机规范中只规定了一个对象的引用，并没有定义这个引用应该通过哪种方式去定位，访问java堆中的具体对象实例，所以一般的访问方式也是取决于java虚拟机的类型，目前主要有两种

- 句柄访问

  使用句柄访问方式，java堆将会划分出来一部分内存去作为句柄池，reference中存储的就是对象的句柄地址，而句柄中则包含对象实例数据的地址和对象类型数据（如对象的类型，实现的接口，方法，父类，field等）的具体地址信息，下边我以一个例子来简单的说明一下

  ​	object object  = new Object();

  object 表示一个本地引用，存储在java栈的本地变量表中，表示一个reference类型的数据

  new Object()作为实例对象存放在java堆中，同时java堆中还存储了Object类的信息（对象类型、实现接口、方法等）的具体地址信息，这些地址信息所执行的数据类型存储在方法区中。

- 直接指针访问

  如果使用直接指针访问，那么java堆对象的布局中就必须考虑如何放置访问类型的相关信息，而reference中存储的就是对象的地址

- 优缺点比较

  - 句柄访问的最大好处是reference中存储着稳定的句柄地址，而对象移动之后，只需要改变句柄中对象的实例地址即可，reference不用改变
  - 使用指针的最大好处是访问速度快，它减少了一次指针定位的时间开销，由于java是面向对象的语言，在开发中java对象的访问非常频繁，因此此类开销积少成多是非常可观的，反之则提升访问速度。

## java是解释性语言还是编译型语言

一、你可以说它是编译型的。因为所有的Java代码都是要编译的，.java不经过编译就什么用都没有。 
二、你可以说它是解释型的。因为java代码编译后不能直接运行，它是解释运行在JVM上的，所以它是解释运行的，那也就算是解释的了。 
三、但是，现在的JVM为了效率，都有一些JIT优化。它又会把.class的二进制代码编译为本地的代码直接运行，所以，又是编译的。

## jvm执行java程序的过程

- 将.java文件编译成.class文件并发送到java虚拟机。加载后的java类会被存放在方法区中。虚拟机执行编译器放在class文件中的字节码。
- JVM 加载 class 文件的原理机制
  - JVM 中类的装载是由类加载器(ClassLoader)和它的子类来实现的，Java 中的类加载器是 一个重要的 Java 运行时系统组件，它负责在运行时查找和装入类文件中的类。
  - 由于 Java 的跨平台性，经过编译的 Java 源程序并不是一个可执行程序，而是一个或多个类文件。 当 Java 程序需要使用某个类时，JVM 会确保这个类已经被加载、连接(验证、准备和解析)和 初始化。
  - 类的加载是指把类的.class 文件中的数据读入到内存中，通常是创建一个字节数组读 入.class 文件，然后产生与所加载类对应的 Class 对象。加载完成后，Class 对象还不完整，所以此时的类还不可用。当类被加载后就进入连接阶段，这一阶段包括验证、准备(为静态变量分 配内存并设置默认的初始值)和解析(将符号引用替换为直接引用)三个步骤。最后 JVM 对类 进行初始化，包括:1)如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类; 2)如果类中存在初始化语句，就依次执行这些初始化语句。
  - 类的加载是由类加载器完成的，类加载器包括:根加载器(BootStrap)、扩展加载器(Extension)、 系统加载器(System)和用户自定义类加载器(java.lang.ClassLoader 的子类)