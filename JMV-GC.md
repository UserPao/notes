# 垃圾回收机制

## 判断对象是否是垃圾的算法

1. 引用计数算法

   通过判断对象的引用数量来决定对象是否可以被回收

   每个对象实例都有一个引用计数器，被引用 + 1，完成引用 - 1

   引用计数为 0 的判断为垃圾

   优点 ： 执行效率高，程序执行受影响较小

   缺点： 无法检测出循环引用的情况，导致内存泄漏

2. 可达性分析算法（主流）

   通过判断对象的引用链是否可达来判断对象是否可以被回收

   从GC Root开始扫描，可达的标记为可达，不可达的就被判定为垃圾对象

   > 可以作为GC  Root的对象

   ​			虚拟机栈中引用的对象（栈帧中的本地变量表）

   ​			方法区中的常量引用的对象 

   ​			方法区中的类静态属性引用的对象

   ​			本地方法栈中的JNI（Native方法）的引用对象

   ​			活跃线程的引用对象

## 垃圾回收算法

### 标记-清除算法（Mark and Sweep）

+ 标记

  从根集合进行扫描，对存活的对象进行标记

+ 清除

  对堆内存从头到尾进行线性遍历，回收不可达对象内存

+ 缺点

  - 碎片化

### 复制算法（Copying）

+ 分为对象面和空闲面

  将可用内存按容量和一定比例划分成两块或者多个块，并选择一块或者两块作为对象面，其他作为空闲面

+ 对象在对象面上创建

+ 存活的对象被从对象面复制到空闲面

  当对象面的内存用完了以后，就将对象面上的还存活的对象复制到空闲面上。

+ 将对象面所有对象内存清除

+ 试用场景

  对象存活率低的场景，如年轻代

+ 解决了碎片化的问题

+ 顺序分配内存，简单高效

### 标记-整理算法（Compacting）

+ 标记

  从根节点进行扫描，对存活的对象进行标记

+ 清除

  移动所有存活的对象，且按照内存地址依次排序，然后将末端内存地址以后的内存全部回收。

+ 避免了内存的不连续性

+ 不用设置两块内存互换

+ 适用于存活率较高的场景，如老年代

###  分代收集算法（Generational Collector）

+ 垃圾回收算法的组合

+ 按照对象生命周期的不同划分区域以采用不用的垃圾回收算法

+ 提高了JVM的垃圾回收效率

+ jdk6、jdk7

  ![1579076878392](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579076878392.png)

+ jdk8及以后版本

  ![1579076922974](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579076922974.png)

  #### GC的分类

  ![1579078051011](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579078051011.png)

  + Minor GC （年轻代的GC算法） —— 复制算法

    * Eden区

      对象刚被创建出来的时候，

    * 两个Survivor区

      + form

      * toSurvivor

  + Major GC（老年代的GC算法） —— 标记-整理算法（Compacting）

  + Full GC（年轻代和老年代一起GC）

    + 慢，执行频率低
    
    + 触发的条件
      + 老年代空间不足
      + 永久代空间不足（JDK7以及以前的版本），JDK8以后使用元空间替代永久代，降低Full GC的频率，减少GC负担
      + CMS GC 时出现promotion failed，concurrent mode failure
      + Minor GC 晋升到老年代的平均大小大于老年代的剩余空间
      + 调用System.gc()
      + 使用RMI(远程方式)来进行RPC或管理的JDK应用，每小时执行一次Full GC 
      
    + **full gc次数过多的优化方式** 
    
      + System.gc()方法的调用：
    
        + 此方法的调用是建议JVM进行Full GC,虽然只是建议而非一定,但很多情况下它会触发 Full GC,从而增加Full GC的频率,也即增加了间歇性停顿的次数。强烈影响系统，建议能不使用此方法就别使用，让虚拟机自己去管理它的内存，可通过通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc。
    
      + 老年代空间不足
    
        + 老年代空间只有新生代对象转入以及创建大对象，大数组的时候才会出现不足的现象，当执行full gc后空间仍不足的，则抛出错误，为了避免以上两种情况出现full gc，调优的时候尽量做到让对象在Minor GC阶段被回收，让对象在新生代多存活一段时间以及不要创建过大的对象以及数组
    
      + **永久区空间不足 **
    
        + JVM规范中运行时数据区域中的方法区，在HotSpot虚拟机中又被习惯称为永久代或者永生代或者永升区。方法区中存放的为一些class的信息，常量，静态变量等数据，当系统中要加载的类，反射的类和调用的方法较多时，方法区可能会被占满，在未配置为采用CMS GC的情况下也会执行Full GC，如果经过Full GC仍然回收不了，那么JVM会抛出错误
    
          为避免方法区占满造成Full GC现象，可采用的方法为增大方法区空间或转为使用CMS GC。
    
      + ### CMS GC时出现promotion failed和concurrent mode failure
    
        + 对于采用CMS进行老年代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。
          promotion failed是在进行Minor GC时，survivor 区放不下、对象只能放入老年代，而此时老年代也放不下造成的；concurrent mode failure是在执行CMS GC的同时有对象要放入老年代，而此时老年代空间不足造成的（有时候“空间不足”是CMS GC时当前的浮动垃圾过多导致暂时性的空间不足触发Full GC）；
          对应解决办法为：增大survivor space、老年代空间或调低触发并发GC的比率。
    
      + ### 统计得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间
    
        + 这是一个较为复杂的触发情况，Hotspot为了避免由于新生代对象晋升到老年代导致老年代空间不足的现象，在进行Minor GC时，做了一个判断，如果之前统计所得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间，那么就直接触发Full GC。
          例如程序第一次触发Minor GC后，有6MB的对象晋升到老年代，那么当下一次Minor GC发生时，首先检查老年代的剩余空间是否大于6MB，如果小于6MB，则执行Full GC。
    
      + ### 堆中分配很大的对象
    
        + 所谓大对象，是指需要大量连续内存空间的java对象，例如很长的数组，此种对象会直接进入老年代，而老年代虽然有很大的剩余空间，但是无法找到足够大的连续空间来分配给当前对象，此种情况就会触发JVM进行Full GC。
    
          为了解决这个问题，CMS垃圾收集器提供了一个可配置的参数，即-XX:+UseCMSCompactAtFullCollection开关参数，用于在“享受”完Full GC服务之后额外免费赠送一个碎片整理的过程，内存整理的过程无法并发的，空间碎片问题没有了，但停顿时间不得不变长了，JVM设计者们还提供了另外一个参数 -XX:CMSFullGCsBeforeCompaction，这个参数用于设置在执行多少次不压缩的Full GC后，跟着来一次带压缩的。

+ 对象如何晋升到老年代

  + 经历一定Minor次数依然存活的对象，默认是15

  + Survivor区放不上下的对象
  + 新生成的大对象（-XX:PretenuerSizeThreshold）

+ 常用的调优参数

  + -XX:SurvivorRatio: Eden 和Survivor的比值，默认是8:1
  + -XX:NewRatio：老年代和年轻代内存大小的比例
  + -XX:MaxTenuringThreshold：对象从年轻代晋升到老年代经过GC次数的最大阈值

### 新生代垃圾回收器

#### Serial收集器

​	-XX:+ UseSerialGC，复制算法

- 单线程收集，进行垃圾收集时，必须暂停所有工作线程
- 简单高效，Client模式下默认的年轻代收集器

#### ParNew收集器

​	-XX:+ UseParNewGC，复制算法

- 多线程收集，其余的行为、特点和Serial收集器一样
- 单核执行效率不如Serial，多核下执行有优势

#### Paraller Scavenge收集器

​	-XX : + UseParallerGC，复制算法

​	吞吐量：运行用户代码时间/ (运行用户代码时间 + 垃圾收集时间)

- 比起关注用户线程停顿时间，更关注系统吞吐量，高效率利用cpu
- 在多核下执行才有优势，Server模式下默认的年轻代收集器

### 老年代垃圾回收器

#### Serial Old 收集器

​	-- XX: + UseSerialOldGC，标记-整理算法

- 单线程收集，进行垃圾收集时，必须暂停所有的工作线程
- 简单高效，Client模式下默认的老年代收集器

#### Paraller Old收集器

​	-XX： + UseParrallelOldGC，标记-整理算法

- 多线程，吞吐量优先

#### CMS 收集器【重要】并发标记清除收集器

​	-XX： + UseConcMarkSweepGC，标记-清除算法

- **过程**
  
  - 初始标记   初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
  - 并发标记：并发追溯标记，程序不会停顿
  - 重新标记：重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短，仍然需要“Stop The World”。
  - 并发清理：清理垃圾对象，程序不会卡顿
  - 并发重置：重置CMS收集器的数据结构
  
- 优点

  并发收集,低停顿

- 缺点

  - CMS收集器对cpu资源非常敏感，其实面向并发设计的程序都对cpu资源比较敏感。在并发阶段，他虽然不会导致用户线程停顿，但是会因为占用了一部分线程而导致应用程序变慢。总吞吐量会降低

  - CMS收集器无法处理浮动垃圾

    CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生

    由于CMS并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，只好留待下一次GC时再清理掉。这一部分垃圾就称为“浮动垃圾”
  
  - 由于并发进行，CMS在收集与应用线程会同时会增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW的方式进行一次GC，从而造成较大停顿时间；
  
  - 标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数-XX:CMSFullGCsBeForeCompaction(默认0，即每次都进行内存整理)来指定多少次CMS收集之后，进行一次压缩的Full GC。
    
  

#### G1收集器【最重要】

​	-XX： + UseG1GC，复制+标记-整理算法

- G1收集器对堆空间的划分

  ![1589955805271](E:\文档\学习资料\笔记\面经\JMV-GC.assets\1589955805271.png)

  **其中每个region（分区）的大小都是在1M-32M之间，并且都是2的幂次方**，启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小，默认是划分出2018个块。

  **目前Region的大小很小，假如一个对象的大小大于一个Region的话如何分配空间**

  1. **0.5Region <= x <  1Region   直接放在old区域，并且old区域标记成H（超大对象存储区)区域**

  2. **1Region < x    申请两个Region，并且标记这两个Region都为H区域**

     ------------------------

  - **RSet**

    每一个Region都会有一个区域存储RSet，**Rset中存储其他Region中引用当前Region中对象的记录**

  - **CSet**

    表示本次垃圾收集需要清理的Region的集合

- **G1的收集过程**

  - **YGC**

    在年轻代的收集过程叫YGC，由于年轻代分为Eden区和survivor from 和survivor to区，在GC的过程中运用**复制算法**将Eden区和survivor from需要存活的对象拷贝到survivor to区。

  - **Mix GC**

    在老年代收集的时候，也同时会把年轻代进行收集，所以叫做Mix GC。过程如下

    1. **初始标记(stop the word)**，标记GC Root对象，**包括GC Root对象所在的Region，叫做RootRegion**，
    2. **通过得到的RootRegion去扫描整个old区的所有Region，看这些Region的RSet中是否有Root Region，**
    3. **并发标记**，遍历那些Region的RSet中有RootRegion的看是否能找到。
    4. **重新标记(stop the word)**，和CMS相似，不过使用的新的算法叫**SATB**，比CMS更快
    5. **清理(stop the word)**,采用复制清理的算法,对于年轻代的Region进行全选，**对于老年代的Region只选择垃圾多的Region**

### 常见垃圾回收期之间的关系

<!-- 有连线表示可以共存-->

![1579082207072](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579082207072.png)

# GC 常见面试题

> Object的finalize方法的作用是否与c++的析构函数相同

~~~shell
1. 不同，析构函数调用确定，而finalize是不确定的
2. 将未被引用的对象置于F-queue队列
3. 方法执行随时可能被终止
4. 给与对象最后一次存活的机会
~~~

> Java中的强引用，软引用，弱引用，虚引用

~~~shell
1. 强引用（Strong Reference）
	1. 最普遍的引用: Object obj = new Object()
	2. 宁愿抛出OutOfMemoryError终止程序也不愿回收具有强引用的对象
	3. 通过将对象置为null来弱化引用，使其被回收
2. 软引用（Soft Reference）
	1. 对象处于有用但是非必须的状态
	2. 只有当内存空间不足的时候，GC会回收该引用的对象的内存
	3. 可以用来实现高速缓存
	4. 
		String str = new String("abc");  // 强引用
		SoftReference<String> softRef = new SoftReference<String>(str); // 软引用
3. 弱引用（week Reference）
	1. 非必须的对象，对软引用更弱
	2. GC时会被回收
	3. 被回收的概率不大，因为GC线程优先级较低
	4. 适用于引用偶尔被使用且不影响垃圾收集的对象
	5.
		String str = new String("abc");  // 强引用
		WeekReference<String> softRef = new WeekReference<String>(str); // 弱引用
4. 虚引用（PhantomReference）
	1. 不会决定对象的生命周期
	2. 任何时候都可能被垃圾回收期回收
	3. 跟踪对象被垃圾回收器回收的活动，起哨兵的作用
	4. 必须和引用队列ReferenceQueue联合使用
	5. 
		String str = new String("abc");
		ReferenceQueue queue = new ReferenceQueue();
		PhantomReference ref = new PhantomReference(str, queue);

强引用 > 软引用 > 弱引用 > 虚引用

引用队列
1. 无实际存储结构，存储逻辑依赖于内部节点之间的关系来表达
2. 存储关联的且被GC的软引用，弱引用，以及虚引用
~~~

![1579083850331](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1579083850331.png)

