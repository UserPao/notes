## 封装

- 定义

  ​		把类内部的信息统一保护起来，防治外包随意修改内部数据，保证数据的安全性，保证外部尽可能正确的使用这个类

- **优点**

  ​		**模块化，项目可以分工合作，便于项目的维护，安全，代码利用率高**

- 缺点

  ​		效率低。

## 多态

**指一类事物有多种状态，比如动物有人，猪，狗等多种状态**

- 优点

  - 增加了程序的灵活性
  - 增加了程序的可扩展性

- **多态的实现原理**

  ​		在JVM执行Java字节码的时候，**类型信息就被存储在方法区中**，通常**为了优化对象调用方法的速度，方法区的类型信息中都增加一个指针，该指针指向一张记录该类方法入口的表（称为方法表），表中每一项都是指向对应方法的指针。**

  方法表的构造如下：

  ​		由于java是单继承，一个类只能继承一个父类，而所有的类又都继承自Object类。方法表中最现存的是Object类的方法，接下来是父类的方法，最后是这个类的方法。**关键的是，如果子类改写了父类方法，那么子类和父类的那些同名方法共享一个方法表项，都被认作是父类的方法。**

  ​		注意这里只有非私有的实例方法才会出现，并且静态方法也不会出现在这里，原因很容易理解：静态方法跟对象无关，可以将方法地址直接引用，而不像实例方法需要间接引用。

  ​		更深入地讲，静态方法是由虚拟机指令invokestatic调用的，私有方法和构造函数则是由invokespecial指令调用，只有被invokevirtual和invokeinterface指令调用的方法才会在方法表中出现。

  ​		由于以上方法的排列特性（Object——父类——子类），使得方法表的偏移量总是固定的。例如，对于任何类来说，其方法表中equals方法的偏移量总是一个定值，所有继承某父类的子类的方法表中，其父类所定义的方法的偏移量也总是一个定值。

  ​		**假设Class A是Class B的子类，并且A改写了B的方法method()，那么在B的方法表中，method方法的指针指向的就是B的method方法入口。**

  ​		**而对于A来说，它的方法表中的method方法则会指向其自身的method方法而非其父类的（这在类加载器载入该类时已经保证，同时JVM会保证总是能从对象引用指向正确的类型信息）。**

- **多态机制包括静态多态（编译时多态）和动态多态（运行时多态），静态多态比如说重载，动态多态是在编译时不能确定调用哪个方法，得在运行时确定。动态多态的实现方法包括子类继承父类和类实现接口。当多个子类上转型（不知道这么说对不）时，对象掉用的是相应子类的方法，这种实现是与JVM有关的。**

- 在调用方法时，实际上必须首先完成实例方法的符号引用解析，结果是该符号引用被解析为方法表的偏移量。虚拟机通过对象引用得到方法区中类型信息的入口，查询类的方法表，当将子类对象声明为父类类型时，形式上调用的是父类方法，此时虚拟机会从实际类的方法表（虽然声明的是父类，但是实际上这里的类型信息中存放的是子类的信息）中查找该方法名对应的指针（这里用“查找”实际上是不合适的，前面提到过，方法的偏移量是固定的，所以只需根据偏移量就能获得指针），进而就能指向实际类的方法了。

- 事实上上面的过程仅仅是利用继承实现多态的内部机制，多态的另外一种实现方式：实现接口相比而言就更加复杂，原因在于，Java的单继承保证了类的线性关系，而接口可以同时实现多个，这样光凭偏移量就很难准确获得方法的指针。所以在JVM中，多态的实例方法调用实际上有两种指令：

  - invokevirtual指令用于调用声明为类的方法；
  - invokeinterface指令用于调用声明为接口的方法。

  当使用invokeinterface指令调用方法时，就不能采用固定偏移量的办法，只能老老实实挨个找了（当然实际实现并不一定如此，JVM规范并没有规定究竟如何实现这种查找，不同的JVM实现可以有不同的优化算法来提高搜索效率）。我们不难看出，在性能上，调用接口引用的方法通常总是比调用类的引用的方法要慢。这也告诉我们，在类和接口之间优先选择接口作为设计并不总是正确的

## JAVA中的集合

- **List**

  List中的对象按照索引位置排序，可以有重复对象，允许按照对象在集合中的索引位置检索对象，如通过list.get(i)方式来获得List集合中的元素。

- **Map**

  Map中的每一个元素包含一个键对象和值对象，它们成对出现。键对象不能重复，值对象可以重复。

- **Set**

  Set中的对象不按特定方式排序，并且没有重复对象。但它的有些实现类能对集合中的对象按特定方式排序，例如TreeSet类，它可以按照默认排序，也可以通过实现java.util.Comparator<Type>接口来自定义排序方式。
## final关键字的一些总结

- 对于引用类型的变量，在对其初始化后便不能再让他指向另一个对象
- 修饰类时，表明这个类不能被继承，final类中的所有成员方法都会被隐式的指定为final方法
- 早期的java版本中，将final方法转为内嵌调用，如果方法过于庞大，则看不到任何性能提升。类中的所有private方法都隐式的指定为final

## 反射

java类的执行需要经历以下过程：

- 编译:.java文件编译后生成.class字节码文件
- 加载：类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例
- 连接：细分三步
  -   验证：格式（class文件规范） 语义（final类是否有子类） 操作
  -   准备：静态变量赋初值和内存空间，final修饰的内存空间直接赋原值，此处不是用户指定的初值。
  -   解析：符号引用转化为直接引用，分配地址
- 初始化:有父类先初始化父类，然后初始化自己；将static修饰代码执行一遍，如果是静态变量，则用用户指定值覆盖原有初值；如果是代码块，则执行一遍操作。

**Java的反射就是利用上面第二步加载到jvm中的.class文件来进行操作的。.class文件中包含java类的所有信息，当你不知道某个类具体信息时，可以使用反射获取class，然后进行各种操作。**

Java反射就是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；并且能改变它的属性。总结说：反射就是把java类中的各种成分映射成一个个的Java对象，并且可以进行操作。

## Object类的方法

- Sleep方法没有释放锁，而wait方法释放了锁，timeout是等待时间

## equals()和hashCode()

首先说**equals相等的两个对象，hashCode一定相等，hashCode相等的对象equals不一定相等**

- equals

  反应的是对象或者变量的具体的值，即两个对象里面包含的值

- hashCode

  计算出对象实例的哈希码，并返回哈希码，又称为散列函数

- **为什么重写equals的时候总是要重写hashCode**

  如果两个对象根据equals方法比较返回的值是相等的，则调用这两个对象的hashCode就要返回相同的数字

- **重写equals而没有重写hashCode会怎么样**

  **如果对象不会放在HashSet等散列相关的集合类中，不会有影响**

  对于存入hashSet的对象，判断的时候会**先按照hashCode值判断，如果不相同，立即判定为不是同一个对象，如果，如果相同才会判断equals是否相等**

  也就是说，HashSet可能会存入两个相同的元素

## 一个中文占几个字节

- ASCII：**一个英文字母占一个字节的空间，一个中文汉字占两个字节的空间**
- UTF-8：**一个英文字符等于一个字节，一个中文汉字占三个字节的空间**
- Unicode编码：**一个英文字符等于两个字节，一个中文等于两个字节**

## 四种情况下，finally块不会被执行

- 在finally语句块中发生了异常
- 在前面的代码中用了system.exit()退出程序
- 程序所在的线程死亡
- 关闭CPU

## 两种键盘输入方法

```java
Scanner input = new Scanner（System.in）;
String s = input.nextLine();
input.close();
```

```java
BufferedReader input  = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```

## 接口和抽象类的区别

- 接口的方法默认是public，所有方法在接口中不能有实现（Java 8开始接口方法可以有默认实现），抽象类可以有非抽象的方法
- 接口中的实例变量默认是final类型的，而抽象类则不一定
- 一个类可以实现多个接口，但最多只能实现一个抽象类
- 一个类实现接口的话要实现所有的方法，但继承抽象类不需要
- 接口不能new实例化，但可以声明，但是必须引用一个实现该接口的对象，从设计层面看，抽象是对类的抽象，是一种设计模式，接口是行为的抽象，是一种行为的规范

应用场景

- 抽象类

  规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能

  其下所有子类都应该有该方法但是大部分子类具体的执行步骤是有所不同的。

- 接口

  需要实现特定的多项功能，而这些功能之间可能完全没有任何联系。

## 继承类和实现接口

适用继承的场景一般是**为了给实体类添加额外属性或者提供共同属性而创建一个基类，这时实体类继承这个基类，就有了额外的属性或者共同的属性**

**适用实现的场景是需要实现特定的多项功能，而且这些功能是没有联系的**

##  泛型擦除

### 类型上界

传递的参数要是某个类的子类。

~~~java
public class Generic1<T extends List<String>> {
    T t;
    List<T> list;

    public <K extends Number> K test(K e) {
        return e;
    }

    public static void main(String[] args) {
        Generic1<ArrayList<String>> g = new Generic1<>();
        System.out.println(g.test((byte) 2));
        System.out.println(g.test(2));
        System.out.println(2L);
        System.out.println(g.test(2.0f));
        System.out.println(g.test(2.0));
        //无法编译，提示参数类型错误
        //System.out.println(g.test("hello"));
    }
}
~~~

### 类型下界

表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object。

~~~java
class Fruit {}

class Apple extends Fruit {}

class Banna extends Fruit {}

class FujiApple extends Apple {}

public class Generic2 {
    public static void test(List<? super FujiApple> list) {
        list.add(new FujiApple());
        //list.add(new Apple());编译错误
    }
}
~~~

### 通配符

？叫做通配符，表示任意类型，上面的例子中已经出现了。它与类型参数T的不同点如下:

- T 只有extends一种限定方式，<T extends List>是合法的，<T super List>是不合法的
- ？有extends与super两种限定方式，即<? extends List> 与<? super List>都是合法的
- T 用于泛型类和泛型方法的定义。？用于泛型方法的调用和形参，即下面的用法是不合法的：

### PECS法则
生产者(Producer)使用 extends，消费者(Consumer)使用 super。

如果需要读取 T 类型的元素，需要声明成 <? extends T>，例如 List<? extends Apple>，此时不能往列表中添加元素。

如果需要添加 T 类型的元素，需要声明成 <？super T>，例如 List<? super Apple>，此时可以向其中添加 Apple 及其子类。从其中取元素的时候，要注意取出元素的类型是 Object。

如果需要同时添加和使用，不使用泛型通配符。

###  泛型擦除

Java中的泛型擦除是指在编译后的字节码文件中类型信息被擦除，变为原生类型(raw type)，因此在运行期，ArrayList<Integer> 与 ArrayList<String> 就是同一个类。

实际上 Java 泛型的擦除并不是对所有使用泛型的地方都会擦除的，部分地方会保留泛型信息。泛型技术相当于 Java 语言的一颗**语法糖**，这种实现泛型的方法称为**伪泛型**

在泛型类被类型擦除的时候，如果类型参数部分没有指定上限，如 <T> 会被转译成普通的 Object 类型，如果指定了上限，则类型参数被替换成类型上限。

例如，**下面的例子在编译期无法通过**：

~~~java
public class Generic4 {
    public void test(ArrayList<Integer> list) {
    }

    public void test(ArrayList<String> list) {
    }
}
~~~

### 泛型与序列化

当序列化一个泛型类，然后反序列化时，会丧失原有的类型信息。

## Error和Exception

​			首先Exception和Error都是继承于Throwable 类，在 Java 中只有 Throwable 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。

​		Exception和Error体现了JAVA这门语言对于异常处理的两种方式。

​		Exception是java程序运行中可预料的异常情况，咱们可以获取到这种异常，并且对这种异常进行业务外的处理。

​		Error是java程序运行中不可预料的异常情况，这种异常发生以后，会直接导致JVM不可处理或者不可恢复的情况。所以这种异常不可能抓取到，比如OutOfMemoryError、NoClassDefFoundError等。
​		其中的Exception又分为运行时异常和非运行时异常。两个根本的区别在于，**运行时异常 必须在编写代码时，使用try catch捕获**（比如：IOException异常）。非运行时异常 在代码编写使，可以忽略捕获操作（比如：ArrayIndexOutOfBoundsException），这种异常是在代码编写或者使用过程中通过规范可以避免发生的。 切记，Error是Throw不是Exception 。

### NoClassDefFoundError 和 ClassNotFoundException 的区别

~~~xml
区别一： NoClassDefFoundError它是Error，ClassNotFoundException是
Exception。

区别二：还有一个区别在于NoClassDefFoundError是JVM运行时通过classpath加载类
时，找不到对应的类而抛出的错误。ClassNotFoundException是在编译过程中如果可能出现此异常，在编译过程中必须将ClassNotFoundException异常抛出！

NoClassDefFoundError发生场景如下：
    1、类依赖的class或者jar不存在 （简单说就是maven生成运行包后被篡改）
    2、类文件存在，但是存在不同的域中 （简单说就是引入的类不在对应的包下)
    3、大小写问题，javac编译的时候是无视大小的，很有可能你编译出来的class文件就与想要的不一样！这个没有做验证


    ClassNotFoundException发生场景如下：
    1、调用class的forName方法时，找不到指定的类
    2、ClassLoader 中的 findSystemClass() 方法时，找不到指定的类

举例说明如下:
    Class.forName("abc"); 比如abc这个类不存项目中，代码编写时，就会提示此异常是检查性异常，比如将此异常抛出。


~~~

### StackOverflowError

~~~java
// 死亡调用
// 报错 Exception in thread "main" java.lang.StackOverflowError
public void cursionDeathLoop(int n){
        cursionDeathLoop(n - 1);
    }
~~~

### OOM

1. java.lang.StackOverflowError

   栈空间溢出，递归调用卡死

2. java.lang.OurOfMemoryError:Java heep space

   堆内存溢出，对象过大

3. java.lang.OutOfMemoryError:GC overhead limit exceeded

   GC回收时间过长，过长的定义为超过98%的时间用来做GC，并且回收了不到2%的堆内存空间，连续多次GC，都回收了不到2%的情况下才会抛出，如果不抛出，那就是GC清理的一点内存很快就会被再次填满，被迫GC再次执行，然后恶性循环

   CPU使用率一直是100%     GC却没有任何成果

   一般是因为堆太小，导致异常的原因：没有足够的内存

   解决方案:

   - 查看代码是否有死循环或者是否有使用大内存的代码

   - 添加JVM的启动参数来限制内存的使用：-XX:-UseGCOverheadLimit

     方法如下：在linux环境下在tomcat的catlina.sh文件中在cygwin= false这一行上面加上

     JAVA_OPTS="-Xms512m -Xmx2048m -Xss1024K -XX:PermSize=256m -XX:MaxPermSize=512m -XX:-UseGCOverheadLimit"

4. java.lang.OutOfMemoryError:Direct buffer memory

   ### 发生原因：

   用来 nio ，但是 direct buffer 不够	

   ### 解决办法

   1）检查是否直接或间接使用了 nio ，例如手动调用生成 buffer 的方法或者使用了 nio 容器如 netty， jetty， tomcat 等等；

   2）-XX:MaxDirectMemorySize 加大，该参数默认是 64M ，可以根据需求调大试试；

   3）检查 JVM 参数里面有无： -XX:+DisableExplicitGC ，如果有就去掉.

5. java.lang.OutOfMemoryError:unable to create new native thread

   **可能原因**

   1. 系统内存耗尽，无法为新线程分配内存
   2. 创建线程数超过了操作系统的限制

   **解决方案**

   1. 排查应用是否创建了过多的线程

      通过jstack确定应用创建了多少线程？超量创建的线程的堆栈信息是怎样的？谁创建了这些线程？一旦明确了这些问题，便很容易解决

   2. 调整操作系统线程数阈值

      操作系统会限制进程允许创建的线程数，使用ulimit -u命令查看限制。某些服务器上此阈值设置的过小，比如1024。一旦应用创建超过1024个线程，就会遇到java.lang.OutOfMemoryError: unable to create new native thread问题。如果是这种情况，可以调大操作系统线程数阈值。

   3. 增加机器内存

      可能是正常增长的业务确实需要更多内存来创建更多线程。如果是这种情况，增加机器内存。

   4. 减小堆内存

      线程不在堆内存上创建，线程在堆内存之外的内存上创建。所以如果分配了堆内存之后只剩下很少的可用内存，依然可能遇到java.lang.OutOfMemoryError: unable to create new native thread。考虑如下场景：系统总内存6G，堆内存分配了5G，永久代512M。在这种情况下，JVM占用了5.5G内存，系统进程、其他用户进程和线程将共用剩下的0.5G内存，很有可能没有足够的可用内存创建新的线程。如果是这种情况，考虑减小堆内存。

   5. 减少进程数

      这和减小堆内存原理相似。考虑如下场景：系统总内存32G，java进程数5个，每个进程的堆内存6G。在这种情况下，java进程总共占用30G内存，仅剩下2G内存用于系统进程、其他用户进程和线程，很有可能没有足够的可用内存创建新的线程。如果是这种情况，考虑减少每台机器上的进程数。

   6.  减小线程栈大小

      线程会占用内存，如果每个线程都占用更多内存，整体上将消耗更多的内存。每个线程默认占用内存大小取决于JVM实现。可以利用-Xss参数限制线程内存大小，降低总内存消耗。例如，JVM默认每个线程占用1M内存，应用有500个线程，那么将消耗500M内存空间。如果实际上256K内存足够线程正常运行，配置-Xss256k，那么500个线程将只需要消耗125M内存。（注意，如果-Xss设置的过低，将会产生java.lang.StackOverflowError错误）  

## 同步异步，阻塞非阻塞

### 同步与异步

**强调的是结果通知的方式，侧重的是被调用者通过何种方式回答我**

**关注的是消息通信机制**

同步：调用一个函数，这个函数在没有计算出来最终结果之前不会返回结果，只有完成了整个函数的执行之后，才会返回最终结果

异步：调用函数，这个函数立即返回，但是没有返回最终结果，当函数执行完成之后，才会将最后结果通知给调用者，比如回调函数。

同步就是一个任务的完成需要依赖另外一个任务时，只有等待被依赖的任务完成后，依赖的任务才能算完成，这是一种可靠的任务序列。要么成功都成功，失败都失败，两个任务的状态可以保持一致。而异步是不需要等待被依赖的任务完成，只是通知被依赖的任务要完成什么工作，依赖的任务也立即执行，只要自己完成了整个任务就算完成了。至于被依赖的任务最终是否真正完成，依赖它的任务无法确定，所以它是不可靠的任务序列。我们可以用打电话和发短信来很好的比喻同步与异步操作。

### 阻塞与非阻塞

**侧重的是调用者，我在等待结果的过程中的状态**

**关注的是程序在等待调用结果(消息，返回值)的状态**

阻塞：调用一个函数，在返回结果之前，调用线程会被挂起，知道最终结果返回才会被唤醒

非阻塞：调用一个函数，在没有返回结果之前并不会阻塞调用线程，调用线程可以去做其他事情，但是要时常过来检查调用结果是否返回。

阻塞与非阻塞主要是从 CPU 的消耗上来说的，阻塞就是 CPU 停下来等待一个慢的操作完成 CPU 才接着完成其它的事。非阻塞就是在这个慢的操作在执行时 CPU 去干其它别的事，等这个慢的操作完成时，CPU 再接着完成后续的操作。虽然表面上看非阻塞的方式可以明显的提高 CPU 的利用率，但是也带了另外一种后果就是**系统的线程切换增加**。增加的 CPU 使用时间能不能补偿系统的切换成本需要好好评估。

非阻塞是相当于维护了一个链表，然后就是无限循环列表查询是否完成了

## IO

- IO的原理

  无论是Socket的读写还是文件的读写，在Java层面的应用开发或者是linux系统底层开发，都属于输入input和输出output的处理，简称为IO读写。在原理上和处理流程上，都是一致的。区别在于参数的不同。

  用户程序进行IO的读写，基本上会用到read&write两大系统调用。可能不同操作系统，名称不完全一样，但是功能是一样的。

  先强调一个基础知识：**read系统调用，并不是把数据直接从物理设备，读数据到内存。write系统调用，也不是直接把数据，写入到物理设备。**

  **read系统调用，是把数据从内核缓冲区复制到进程缓冲区；而write系统调用，是把数据从进程缓冲区复制到内核缓冲区**。这个两个系统调用，都不负责数据在内核缓冲区和磁盘之间的交换。底层的读写交换，是由操作系统kernel内核完成的。

1. BIO

   传统的java.io包，基于流模型实现的，交互的方式是同步，阻塞方式

2. NIO

   NIO 是 Java 1.4 引入的 java.nio 包，提供了 Channel、Selector、Buffer 等新的抽象，可以构建多路复用的、同步非阻塞 IO 程序，同时提供了更接近操作系统底层高性能的数据操作方式

3. AIO

   Java 1.7 之后引入的包，是 NIO 的升级版本，提供了异步非堵塞的 IO 操作方式，所以人们叫它 AIO（Asynchronous IO），异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作
   
   **同步阻塞IO（JAVA BIO）：    同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。**
   
   **同步非阻塞IO (JAVA NIO)： 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。用户进程也需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问。**
   
   **异步阻塞IO ：    此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成，那么为什么说是阻塞的呢？因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄（如果从UNP的角度看，select属于同步操作。因为select之后，进程还需要读写数据），从而提高系统的并发性！ **
   
   **异步非阻塞IO（Java AIO(NIO.2)）:    在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。**  

### NIO

​		是一种**同步非阻塞**的IO模型,主要有三大核心部门Channel(通道),Buffer(缓冲区),Selector(多路复用器)。**传统的IO基于字节流和字符流操作，NIO基于Channel和Buffer进行操作**。数据总是从通道读取到缓冲区，或者从缓冲区写入到通道。**Selector用于监听多个通道的事件（比如：连接打开，数据到达）**，因此**单个线程**可以**监听多个数据通道**。

**优点**

- 通过Channel注册到Selector上的状态来实现一个客户端与服务端的通信
- Channel中数据的读取都是通过Buffer,一种非阻塞的读取方式
- Selector多路复用的单线程模式，线程资源开销相对较小。

**缺点**

- 如果有多个io，需要一个一个检测，每次检测调用read都会发生上下文切换（read是系统调用,每次调用都要在用户态和和心态切换一次)
- 第一次读取不到时，不知道应该等待多久在尝试一次。

### IO多路复用

在多路复用IO模型中，一个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是内核缓冲区可读/可写），内核kernel能够通知程序进行相应的IO系统调用。系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。

IO多路复用有三种方式

1. **Select**

   只有一个函数，调用select的时候，需要将监听句柄和最大等待时间作为参数传递进去，select会发生**阻塞，直到一个事件发生了或者等到最大等待时间就返回**

   - 缺点
     - select 会修改传入的参数数组，这个对于一个需要调用很多次的函数，是非常不友好的。
     - Select返回只返回有准备好的socket，但并不指定是哪个，需要在遍历socket列表。
     - 当用户进程调用了select，那么整个线程会被block（阻塞掉）
     - select是无状态的，即**每次调用select，内核都要重新检查所有被注册的fd的状态**。select返回后，这些状态就被返回了，内核不会记住它们；到了下一次调用，内核依然要重新检查一遍。于是查询的效率很低
     - select能够支持的最大的fd数组的长度是1024
     - select 不是线程安全的，如果你把一个sock加入到select, 然后突然另外一个线程发现，尼玛，这个sock不用，要收回。对不起，这个select 不支持的，如果你丧心病狂的竟然关掉这个sock, select的标准行为是。。呃。。不可预测的， 这个可是写在文档中的哦.

2. **Poll**

   poll优化了select的一些问题，参数变得简单一些，**没有了1024的限制**。但其他的问题依旧。

   - 缺点
     - 依然是无状态
     - 仍然无法直接获取是哪些有事件发生的fd，还是要遍历所有的fd

3. **epoll**

   ​		**epoll可以理解为event poll，不同于忙轮询和无差别轮询，epoll之会把哪个流发生了怎样的I/O事件通知我们**。此时我们对这些流的操作都是有意义的。（复杂度降低到了O(1)）

   ​		epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，**没有描述符限制**。epoll**使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。**		

   - **epoll的主要接口**

     ```java
     int epoll_create(int size);
     int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
     int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
     ```

     - **int epoll_create(int size);**

       创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值。需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

     - **int epoll_ctl(int epfd, int op, int fd, struct epoll_event \*event);**

       epoll的事件注册函数，它不同于select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。

       第一个参数是epoll_create()的返回值，

       第二个参数表示动作，用三个宏来表示：

       - EPOLL_CTL_ADD：注册新的fd到epfd中；
       - EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
       - EPOLL_CTL_DEL：从epfd中删除一个fd；

       第三个参数是需要监听的fd，

       第四个参数是告诉内核需要监听什么事，有以下几种枚举值

       - EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
       - EPOLLOUT：表示对应的文件描述符可以写；
       - EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
       - EPOLLERR：表示对应的文件描述符发生错误；
       - EPOLLHUP：表示对应的文件描述符被挂断；
       - EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
       - EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

     - **int epoll_wait(int epfd, struct epoll_event \* events, int maxevents, int timeout);**

       等待事件的产生，类似于select()调用。参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。

   - **工作模式**

     epoll对文件描述符的操作有两种模式：**LT（level trigger）和ET（edge trigger）**。LT模式是默认模式，LT模式与ET模式的区别如下：

      　　LT模式：**当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。**

      　　ET模式：**当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。**

      　　**ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。**

   - **高效的原因**

     **epoll的高效就在于，当我们调用epoll_ctl往里塞入百万个句柄时，epoll_wait仍然可以飞快的返回，并有效的将发生事件的句柄给我们用户。这是由于我们在调用epoll_create时，内核除了帮我们在epoll文件系统里建了个file结点，在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket外，还会再建立一个list链表，用于存储准备就绪的事件，当epoll_wait调用时，仅仅观察这个list链表里有没有数据即可。有数据就返回，没有数据就sleep，等到timeout时间到后即使链表没数据也返回。所以，epoll_wait非常高效。**

     **而且，通常情况下即使我们要监控百万计的句柄，大多一次也只返回很少量的准备就绪句柄而已，所以，epoll_wait仅需要从内核态copy少量的句柄到用户态而已，如何能不高效？！**

     **那么，这个准备就绪list链表是怎么维护的呢？当我们执行epoll_ctl时，除了把socket放到epoll文件系统里file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪list链表里。所以，当一个socket上有数据到了，内核在把网卡上的数据copy到内核中后就来把socket插入到准备就绪链表里了。**

     **如此，一颗红黑树，一张准备就绪句柄链表，少量的内核cache，就帮我们解决了大并发下的socket处理问题。执行epoll_create时，创建了红黑树和就绪链表，执行epoll_ctl时，如果增加socket句柄，则检查在红黑树中是否存在，存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据。执行epoll_wait时立刻返回准备就绪链表里的数据即可**

**多路复用IO的优点：**

用select/epoll的优势在于，它可以同时处理成千上万个连接（connection）。与一条线程维护一个连接相比，I/O多路复用技术的最大优势是：系统不必创建线程，也不必维护这些线程，从而大大减小了系统的开销。

Java的NIO（new IO）技术，使用的就是IO多路复用模型。在linux系统上，使用的是epoll系统调用。

**多路复用IO的缺点：**

本质上，select/epoll系统调用，属于同步IO，也是阻塞IO。都需要在读写事件就绪后，自己负责进行读写，也就是说这个读写过程是阻塞的。

如何充分的解除线程的阻塞呢？那就是异步IO模型。

### AIO

​		AIO 是 Java 1.7 之后引入的包，是 NIO 的升级版本，提供了异步非堵塞的 IO 操作方式，所以人们叫它 AIO（Asynchronous IO），异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

AIO的基本流程是：用户线程通过系统调用，告知kernel内核启动某个IO操作，用户线程返回。kernel内核在整个IO操作（包括数据准备、数据复制）完成后，通知用户程序，用户执行后续的业务操作。

kernel的数据准备是将数据从网络物理设备（网卡）读取到内核缓冲区；kernel的数据复制是将数据从内核缓冲区拷贝到用户程序空间的缓冲区。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190105163914730.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

（1）当用户线程调用了read系统调用，立刻就可以开始去做其它的事，用户线程不阻塞。

（2）内核（kernel）就开始了IO的第一个阶段：准备数据。当kernel一直等到数据准备好了，它就会将数据从kernel内核缓冲区，拷贝到用户缓冲区（用户内存）。

（3）kernel会给用户线程发送一个信号（signal），或者回调用户线程注册的回调接口，告诉用户线程read操作完成了。

（4）用户线程读取用户缓冲区的数据，完成后续的业务操作。

异步IO模型的特点：

在内核kernel的等待数据和复制数据的两个阶段，用户线程都不是block(阻塞)的。用户线程需要接受kernel的IO操作完成的事件，或者说注册IO操作完成的回调函数，到操作系统的内核。所以说，异步IO有的时候，也叫做信号驱动 IO 。



异步IO模型缺点：

需要完成事件的注册与传递，这里边需要底层操作系统提供大量的支持，去做大量的工作。

目前来说， Windows 系统下通过 IOCP 实现了真正的异步 I/O。但是，就目前的业界形式来说，Windows 系统，很少作为百万级以上或者说高并发应用的服务器操作系统来使用。

而在 Linux 系统下，异步IO模型在2.6版本才引入，目前并不完善。所以，这也是在 Linux 下，实现高并发网络编程时都是以 IO 复用模型模式为主。

------------------------------------------------------------------------------------------------------------------------------

### NIO和IO的区别

1. Non-blocking IO（非阻塞IO）

   **IO流是阻塞的，NIO流是不阻塞的。**

   Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

   Java IO的各种流是阻塞的。这意味着，当一个线程调用 `read()` 或 `write()` 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了

2. Buffer(缓冲区)

   **IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。**

   Buffer是一个对象，它包含一些要写入或者要读出的数据。在NIO类库中加入Buffer对象，体现了新库与原I/O的一个重要区别。在面向流的I/O中·可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而 NIO 却是直接读到 Buffer 中进行操作。

   在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

   最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer,还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。

3. Channel (通道)

   NIO 通过Channel（通道） 进行读写。

   通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。

4. Selectors(选择器)

   NIO有选择器，而IO没有。

   选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

#### NIO为什么比BIO快

假如有10000个连接，4核CPU ，那么bio 就需要一万个线程，而nio大概就需要5个线程(一个接收请求，四个处理请求)。如果这10000个连接同时请求，那么bio就有10000个线程抢四个CPU ，几乎每个CPU 平均执行2500次上下文切换，而nio 四个处理线程，几乎每个线程都对应一个CPU ，也就是几乎没有上下文切换。效率就体现出来了。

## Java 8的新特性

### **Lambda表达式**

> lambda表达式本质上是一段匿名内部类，也可以是一段可以传递的代码

```java
  //匿名内部类
  Comparator<Integer> cpt = new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
          return Integer.compare(o1,o2);
      }
  };

  TreeSet<Integer> set = new TreeSet<>(cpt);

  //使用lambda表达式
  Comparator<Integer> cpt2 = (x,y) -> Integer.compare(x,y);
  TreeSet<Integer> set2 = new TreeSet<>(cpt2);
```

**Lmabda表达式的语法总结： () -> ();**

|                  **前置**                  |                         语法                         |
| :----------------------------------------: | :--------------------------------------------------: |
|               无参数无返回值               |       () -> System.out.println(“Hello WOrld”)        |
|             有一个参数无返回值             |             (x) -> System.out.println(x)             |
|          有且只有一个参数无返回值          |              x -> System.out.println(x)              |
|  有多个参数，有返回值，有多条lambda体语句  | (x，y) -> {System.out.println(“xxx”);return xxxx;}； |
| 有多个参数，有返回值，只有一条lambda体语句 |                    (x，y) -> xxxx                    |

**口诀：左右遇一省括号，左侧推断类型省**

> 注：当一个接口中存在**多个抽象方法**时，如果使用lambda表达式，并不能智能匹配对应的抽象方法，因此引入了函数式接口的概念



### **函数式接口**

> 函数式接口的提出是为了给Lambda表达式的使用提供更好的支持。

定义: 定义了**有且只有一个抽象**方法的接口（Object类的public方法除外），就是函数式接口，并且还提供了注解：@FunctionalInterface

```java
public interface Function {
    public void say();
    default public void say2(){

    }
}
```



常见的四大函数式接口

- Consumer 《T》：消费型接口，有参无返回值

  ```java
      @Test
      public void test(){
          changeStr("hello",(str) -> System.out.println(str));
      }
  
      /**
       *  Consumer<T> 消费型接口
       * @param str
       * @param con
       */
      public void changeStr(String str, Consumer<String> con){
          con.accept(str);
      }
  ```

- Supplier 《T》：供给型接口，无参有返回值

  ```java
      @Test
      public void test2(){
          String value = getValue(() -> "hello");
          System.out.println(value);
      }
  
      /**
       *  Supplier<T> 供给型接口
       * @param sup
       * @return
       */
      public String getValue(Supplier<String> sup){
          return sup.get();
      }
  ```

- Function 《T,R》：:函数式接口，有参有返回值

  ```java
      @Test
      public void test3(){
          Long result = changeNum(100L, (x) -> x + 200L);
          System.out.println(result);
      }
  
      /**
       *  Function<T,R> 函数式接口
       * @param num
       * @param fun
       * @return
       */
      public Long changeNum(Long num, Function<Long, Long> fun){
          return fun.apply(num);
      }
  ```

- Predicate《T》： 断言型接口，有参有返回值，返回值是boolean类型

  ```java
  public void test4(){
          boolean result = changeBoolean("hello", (str) -> str.length() > 5);
          System.out.println(result);
      }
  
      /**
       *  Predicate<T> 断言型接口
       * @param str
       * @param pre
       * @return
       */
      public boolean changeBoolean(String str, Predicate<String> pre){
          return pre.test(str);
      }
  ```

### **方法引用和构造器调用**

> 若lambda体中的内容有方法已经实现了，那么可以使用“方法引用”
> 也可以理解为方法引用是lambda表达式的另外一种表现形式并且其语法比lambda表达式更加简单

**方法引用三种表现形式：**

1. 对象：：实例方法名
2. 类：：    静态方法名
3. 类：：    实例方法名 （lambda参数列表中第一个参数是实例方法的调用 者，第二个参数是实例方法的参数时可用）

```java
 public void test() {
        /**
        *注意：
        *   1.lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值类型保持一致！
        *   2.若lambda参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，可以使用ClassName::method
        *
        */
        Consumer<Integer> con = (x) -> System.out.println(x);
        con.accept(100);

        // 方法引用-对象::实例方法
        Consumer<Integer> con2 = System.out::println;
        con2.accept(200);

        // 方法引用-类名::静态方法名
        BiFunction<Integer, Integer, Integer> biFun = (x, y) -> Integer.compare(x, y);
        BiFunction<Integer, Integer, Integer> biFun2 = Integer::compare;
        Integer result = biFun2.apply(100, 200);

        // 方法引用-类名::实例方法名
        BiFunction<String, String, Boolean> fun1 = (str1, str2) -> str1.equals(str2);
        BiFunction<String, String, Boolean> fun2 = String::equals;
        Boolean result2 = fun2.apply("hello", "world");
        System.out.println(result2);
    }
```

**构造器引用**

格式：ClassName::new

```java
public void test2() {

        // 构造方法引用  类名::new
        Supplier<Employee> sup = () -> new Employee();
        System.out.println(sup.get());
        Supplier<Employee> sup2 = Employee::new;
        System.out.println(sup2.get());

        // 构造方法引用 类名::new （带一个参数）
        Function<Integer, Employee> fun = (x) -> new Employee(x);
        Function<Integer, Employee> fun2 = Employee::new;
        System.out.println(fun2.apply(100));
 }
```

**数组引用**

格式：Type[]::new

```java
public void test(){
        // 数组引用
        Function<Integer, String[]> fun = (x) -> new String[x];
        Function<Integer, String[]> fun2 = String[]::new;
        String[] strArray = fun2.apply(10);
        Arrays.stream(strArray).forEach(System.out::println);
}
```

### **Stream API**

Stream操作的三个步骤

- 创建stream

  ```java
      // 1，校验通过Collection 系列集合提供的stream()或者paralleStream()
      List<String> list = new ArrayList<>();
      Strean<String> stream1 = list.stream();
  
      // 2.通过Arrays的静态方法stream()获取数组流
      String[] str = new String[10];
      Stream<String> stream2 = Arrays.stream(str);
  
      // 3.通过Stream类中的静态方法of
      Stream<String> stream3 = Stream.of("aa","bb","cc");
  
      // 4.创建无限流
      // 迭代
      Stream<Integer> stream4 = Stream.iterate(0,(x) -> x+2);
  
      //生成
      Stream.generate(() ->Math.random());
  ```

- 中间操作（过滤、map）

  ```java
  /**
     * 筛选 过滤  去重
     */
    emps.stream()
            .filter(e -> e.getAge() > 10)
            .limit(4)
            .skip(4)
            // 需要流中的元素重写hashCode和equals方法
            .distinct()
            .forEach(System.out::println);
  
  
    /**
     *  生成新的流 通过map映射
     */
    emps.stream()
            .map((e) -> e.getAge())
            .forEach(System.out::println);
  
  
    /**
     *  自然排序  定制排序
     */
    emps.stream()
            .sorted((e1 ,e2) -> {
                if (e1.getAge().equals(e2.getAge())){
                    return e1.getName().compareTo(e2.getName());
                } else{
                    return e1.getAge().compareTo(e2.getAge());
                }
            })
            .forEach(System.out::println);
  
  ```

- 终止操作

  ```java
   /**
           *      查找和匹配
           *          allMatch-检查是否匹配所有元素
           *          anyMatch-检查是否至少匹配一个元素
           *          noneMatch-检查是否没有匹配所有元素
           *          findFirst-返回第一个元素
           *          findAny-返回当前流中的任意元素
           *          count-返回流中元素的总个数
           *          max-返回流中最大值
           *          min-返回流中最小值
           */
  
          /**
           *  检查是否匹配元素
           */
          boolean b1 = emps.stream()
                  .allMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
          System.out.println(b1);
  
          boolean b2 = emps.stream()
                  .anyMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
          System.out.println(b2);
  
          boolean b3 = emps.stream()
                  .noneMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
          System.out.println(b3);
  
          Optional<Employee> opt = emps.stream()
                  .findFirst();
          System.out.println(opt.get());
  
          // 并行流
          Optional<Employee> opt2 = emps.parallelStream()
                  .findAny();
          System.out.println(opt2.get());
  
          long count = emps.stream()
                  .count();
          System.out.println(count);
  
          Optional<Employee> max = emps.stream()
                  .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
          System.out.println(max.get());
  
          Optional<Employee> min = emps.stream()
                  .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
          System.out.println(min.get());
  ```

  还有功能比较强大的两个终止操作 **reduce和collect**

  reduce操作： reduce:(T identity,BinaryOperator)/reduce(BinaryOperator)-可以将流中元素反复结合起来，得到一个值

  ```java
           /**
           *  reduce ：规约操作
           */
          List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
          Integer count2 = list.stream()
                  .reduce(0, (x, y) -> x + y);
          System.out.println(count2);
  
          Optional<Double> sum = emps.stream()
                  .map(Employee::getSalary)
                  .reduce(Double::sum);
          System.out.println(sum);
  
  ```

  collect操作：Collect-将流转换为其他形式，接收一个Collection接口的实现，用于给Stream中元素做汇总的方法

  ```java
          /**
           *  collect：收集操作
           */
  
          List<Integer> ageList = emps.stream()
                  .map(Employee::getAge)
                  .collect(Collectors.toList());
          ageList.stream().forEach(System.out::println);
  ```

  

### **接口中的默认方法和静态方法**

在接口中可以使用default和static关键字来修饰接口中定义的普通方法

```java
public interface Interface {
    default  String getName(){
        return "zhangsan";
    }

    static String getName2(){
        return "zhangsan";
    }
}
```

在JDK1.8中很多接口会新增方法，为了保证1.8向下兼容，1.7版本中的接口实现类不用每个都重新实现新添加的接口方法，引入了default默认实现，static的用法是直接用接口名去调方法即可。当一个类继承父类又实现接口时，若后两者方法名相同，则优先继承父类中的同名方法，即“类优先”，**如果实现两个同名方法的接口，则要求实现类必须手动声明默认实现哪个接口中的方法。**

### **新时间日期API**

**LocalDate | LocalTime | LocalDateTime**

新的日期API都是不可变的，更使用于多线程的使用环境中

- ### LocalDate

  从默认时区的系统时钟获取当前的日期时间。不用考虑时区差

- ### LocalTime

  时间戳  1970年1月1日00：00：00 到某一个时间点的毫秒值

- ### LocalDateTime

### optional

- 声明一个空的optional

  Optional<Car> optCar = Optional.empty();

- 依据一个非空值创建Optional 

  Optional<Car> optCar = Optional.of(car);

  如果car是一个null，这段代码会立即抛出一个NullPointerException，而不是等到你
  试图访问car的属性值时才返回一个错误 

- 可接受null的Optional 

  最后，使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional
  对象：
  Optional<Car> optCar = Optional.ofNullable(car);
  如果car是null，那么得到的Optional对象就是个空对象。 

- 使用flatMap重构代码

## JAVA多态

​		JAVA对于方法调用动态绑定的实现主要依赖于方法表，但通过**类引用调用**和**接口引用调用**的实现则有所不同。总体而言，当某个方法被调用时，JVM首先要查找相应的变量池，得到方法的符号引用，并查找调用类的方法表以确定该方法的直接引用，最后才真正调用该方法。

​		JVM运行时结构

![图 1.JVM 运行时结构](Java EE.assets\image003.jpg)

​		当程序运行需要某个类的定义时，载入子系统 (class loader subsystem) 装入所需的 class 文件，并在内部建立该类的类型信息，这个类型信息就存贮在方法区。类型信息一般包括该类的方法代码、类变量、成员变量的定义等等。可以说，类型信息就是类的 Java 文件在运行时的内部结构，包含了改类的所有在 Java 文件中定义的信息。

​		注意到，该类型信息和 class 对象是不同的。class 对象是 JVM 在载入某个类后于堆 (heap) 中创建的代表该类的对象，可以通过该 class 对象访问到该类型信息。比如最典型的应用，在 Java 反射中应用 class 对象访问到该类支持的所有方法，定义的成员变量等等。可以想象，JVM 在类型信息和 class 对象中维护着它们彼此的引用以便互相访问。两者的关系可以类比于进程对象与真正的进程之间的关系。

### Java 的方法调用方式

Java 的方法调用有两类，**动态方法调用**与**静态方法调用**。静态方法调用是指对于类的静态方法的调用方式，是静态绑定的；而动态方法调用需要有方法调用所作用的对象，是动态绑定的。类调用 (invokestatic) 是在编译时刻就已经确定好具体调用方法的情况，而实例调用 (invokevirtual) 则是在调用的时候才确定具体的调用方法，这就是动态绑定，也是多态要解决的核心问题。

JVM 的方法调用指令有四个，分别是 invokestatic，invokespecial，invokesvirtual 和 invokeinterface。前两个是静态绑定，后两个是动态绑定的。本文也可以说是对于 JVM 后两种调用实现的考察。

### 常量池（constant pool）

常量池中保存的是一个 Java 类引用的一些常量信息，包含一些字符串常量及对于类的符号引用信息等。Java 代码编译生成的类文件中的常量池是静态常量池，当类被载入到虚拟机内部的时候，在内存中产生类的常量池叫运行时常量池。

常量池在逻辑上可以分成多个表，每个表包含一类的常量信息，本文只探讨对于 Java 调用相关的常量池表。

CONSTANT_Utf8_info

字符串常量表，该表包含该类所使用的所有字符串常量，比如代码中的字符串引用、引用的类名、方法的名字、其他引用的类与方法的字符串描述等等。其余常量池表中所涉及到的任何常量字符串都被索引至该表。

CONSTANT_Class_info

类信息表，包含任何被引用的类或接口的符号引用，每一个条目主要包含一个索引，指向 CONSTANT_Utf8_info 表，表示该类或接口的全限定名。

CONSTANT_NameAndType_info

名字类型表，包含引用的任意方法或字段的名称和描述符信息在字符串常量表中的索引。

CONSTANT_InterfaceMethodref_info

接口方法引用表，包含引用的任何接口方法的描述信息，主要包括类信息索引和名字类型索引。

CONSTANT_Methodref_info

类方法引用表，包含引用的任何类型方法的描述信息，主要包括类信息索引和名字类型索引。

![图 2. 常量池各表的关系](Java EE.assets\image005.jpg)

可以看到，给定任意一个方法的索引，在常量池中找到对应的条目后，可以得到该方法的类索引（class_index）和名字类型索引 (name_and_type_index), 进而得到该方法所属的类型信息和名称及描述符信息（参数，返回值等）。注意到所有的常量字符串都是存储在 CONSTANT_Utf8_info 中供其他表索引的。

### 方法表与方法调用

​		方法表是动态调用的核心，也是 Java 实现动态调用的主要方式。它被存储于方法区中的类型信息，包含有该类型所定义的所有方法及指向这些方法代码的指针，注意这些具体的方法代码可能是被覆写的方法，也可能是继承自基类的方法。

## String类为什么要设计成不可变的

1. **字符串常量池的需要**

   字符串常量池(String pool, String intern pool, String保留池) 是Java堆内存中一个特殊的存储区域, 当创建一个String对象时,假如此字符串值已经存在于常量池中,则不会创建一个新的对象,而是引用已经存在的对象。

   如下面的代码所示,将会在堆内存中只创建一个实际String对象

   ```java
   String s1 = "abcd";
   String s2 = "abcd";
   ```

   ![img](Java EE.assets\20131113170355750.jfif)

   **假若字符串对象允许改变,那么将会导致各种逻辑错误,比如改变一个对象会影响到另一个独立对象. 严格来说，这种常量池的思想,是一种优化手段.**

2. **允许String对象缓存HashCode**

   Java中String对象的哈希码被频繁地使用, 比如在hashMap 等容器中。

   字符串不变性保证了hash码的唯一性,因此可以放心地进行缓存.这也是一种性能优化手段,意味着不必每次都去计算新的哈希码. 在String类的定义中有如下代码:

   ~~~java
   private int hash;//用来缓存HashCode
   ~~~

3. **安全性**

   String被许多的JAVA类库用来当作参数，例如网络的URL，文件路径path，还有反射机制所需要的的String参数，如果String不是固定不变的，将会引起各种安全隐患，例如

   ~~~java
   boolean connect(String s){
       if(!isSecure(s)){
           throw new SecurityException();
       }
       // 如果在其他地方可以修改String，那么此处就会引起各种预料不到的问题/错误
       causeProblem(s);
   }
   ~~~


## 正则表达式

| 元字符 | 说明                         |
| ------ | ---------------------------- |
| .      | 匹配除换行符以外的任意字符   |
| \w     | 匹配字母或数字或下划线或汉字 |
| \s     | 匹配任意的空白符             |
| \d     | 匹配数字                     |
| \b     | 匹配单词的开发或结束         |
| ^      | 匹配字符串的开始             |
| $      | 匹配字符串的结束             |

| 语法  | 说明             |
| ----- | ---------------- |
| *     | 重复零次或者多次 |
| +     | 重复一次或更多次 |
| ？    | 重复零次或者一次 |
| {n}   | 重复n次          |
| {n,}  | 重复n次或者更多  |
| {n,m} | 重复n次到m次     |

