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

#### CMS 收集器【重要】并发标记清楚收集器

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

 	之前介绍的几组垃圾收集器组合，都有几个共同点：

1. 年轻代、老年代是独立且连续的内存块；
2. 年轻代收集使用单eden、双survivor进行复制算法；
3. 老年代收集必须扫描整个老年代区域；
4. 都是以尽可能少而块地执行GC为设计原则。

- 特点
  - 并发和并行
    - 使用多个cpu来缩短stop-the-world的时间，与用户线程并发执行
  - 分代收集
  - 空间整合
  - 可预测的停顿
- 将整个java堆内存划分为多个大小相等的Region
- 年轻代和老年代不在物理隔离

**G1也有类似CMS的收集动作：初始标记、并发标记、重新标记、清除、转移回收**

G1收集与以上三组收集器有很大不同：

1. G1的设计原则是"首先收集尽可能多的垃圾(Garbage First)"。因此，G1并不会等内存耗尽(串行、并行)或者快耗尽(CMS)的时候开始垃圾收集，而是在内部采用了启发式算法，在老年代找出具有高收集收益的分区进行收集。同时G1可以根据用户设置的暂停时间目标自动调整年轻代和总堆大小，暂停目标越短年轻代空间越小、总空间就越大；
2. G1采用内存分区(Region)的思路，将内存划分为一个个相等大小的内存分区，回收时则以分区为单位进行回收，存活的对象复制到另一个空闲分区中。由于都是以相等大小的分区为单位进行操作，因此G1天然就是一种压缩方案(局部压缩)；
3. G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别，也不需要完全独立的survivor(to space)堆做复制准备。G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换；
4. G1的收集都是STW的，但年轻代和老年代的收集界限比较模糊，采用了混合(mixed)收集的方式。即每次收集既可能只收集年轻代分区(年轻代收集)，也可能在收集年轻代的同时，包含部分老年代分区(混合收集)，这样即使堆内存很大时，也可以限制收集范围，从而降低停顿。

##### 分区

G1采用了分区(Region)的思路，将整个堆空间分成若干个大小相等的内存区域，每次分配对象空间将逐段地使用内存。因此，在堆的使用上，G1并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可；每个分区也不会确定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小(1MB~32MB，且必须是2的幂)，默认将整堆划分为2048个分区。

##### 卡片

在每个分区内部又被分成了若干个大小为512 Byte卡片(Card)，标识堆内存最小可用粒度所有分区的卡片将会记录在全局卡片表(Global Card Table)中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象(见RSet)。每次对内存的回收，都是对指定分区的卡片进行处理。

##### 堆

G1同样可以通过-Xms/-Xmx来指定堆空间大小。当发生年轻代收集或混合收集时，通过计算GC与应用的耗费时间比，自动调整堆空间大小。如果GC频率太高，则通过增加堆尺寸，来减少GC频率，相应地GC占用的时间也随之降低；目标参数-XX:GCTimeRatio即为GC与应用的耗费时间比，G1默认为9，而CMS默认为99，因为CMS的设计原则是耗费在GC上的时间尽可能的少。另外，当空间不足，如对象空间分配或转移失败时，G1会首先尝试增加堆空间，如果扩容失败，则发起担保的Full GC。Full GC后，堆尺寸计算结果也会调整堆空间。

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
