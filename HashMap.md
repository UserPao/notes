## tableSizeFor()

计算最小的大于等于这个容量的值【2的幂】

~~~java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
~~~

## 下标计算方式

~~~java
index = (n - 1) & hash
~~~

## hash函数的重写

~~~java
return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
~~~

## 不同版本的HashMap的hash方法源码

- 1.8

  ```java
  static final int hash(Object key) {
      int h ;
      // ^ 按位异或，
      // >>> 无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 :(h = key.hashCode()) ^ (h >>> 16);
  }
  ```

- 1.7

  ```java
  static int hash(int h) {
      h ^= (h >>> 20) ^ (h >>> 12);
      return h^ (h >>> 7) ^ (h >>> 4);
  }
  ```

## 

## 内部各个数字限制

1. DEFAULT_INITIAL_CAPACITY = 1 << 4 ； 16

   默认的map长度

2. MAXIMUM_CAPACITY = 1 << 30

   限制最大的map长度

3. DEFAULT_LOAD_FACTOR = 0.75f

   默认的加载因子，当map的长度超过通过map长度*加载因子时，进行扩容

4. TREEIFY_THRESHOLD = 8

   链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树

   之所示是8是因为：理性情况下随机hashcode算法下所有bin节点的分布遵循泊松分布，一个bin中链表长度到达8个元素的概率为0.000000000000006，几乎是不可能事件，所以选择了8

5. UNTREEIFY_THRESHOLD = 6

    红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表

6. MIN_TREEIFY_CAPACITY = 64

   当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树） 否则，若桶内元素太多时，则直接扩容，而不是树形化 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD

## 线程不安全的方式

1. 多线程put：两个计算出相同hash值的数据，后边的会覆盖掉前边的
2. 多线程get:可能会因为扩容陷入死循环：因为是链表模式，可能导致节点的指针循环，导致死循环。jdk8以后解决了。

## hashMap和HashTable的比较

1. table不允许键或者是值是null，map允许键或者是值是null

2. map值为null的时候返回的是0，而table是安全失败机制的，这种机制可能导致你此次读到的数据不是最新的，使用null无法判断对应的key是不存在还是空【安全失败机制：在进行遍历前，把数组复制一份，读取复制的数组，坏处是可能会读到脏数据。】

3. table继承Dcitionary（jdk10新增）类，map继承AbstractMap类

4. map初始容量是16，table初试容量是11，二者的加载因子都是0.75

5. 扩容机制：map是当前容量翻倍，table是当前容量翻倍 + 1【为了保证奇数】

6. hashTable是线程安全的，hashMap不是线程安全的。

7. 迭代器失效机制

   map中的Iterator迭代器是快速失败的【快速失败机制：遍历时如果数组变化，直接抛出异常Concurrent Modiication Exception】，而table的Enumerator是安全失败的【安全失败机制：在进行遍历前，把数组复制一份，读取复制的数组，坏处是可能会读到脏数据。】

> - 快速失败：
>
>   ​		在用迭代器遍历集合对象时，如果遍历过程中对集合内容进行了修改修改（增加、删除、修改），则会抛出Concurrent Modiication Exception
>
>   ​		原理：在遍历的时候维护一个modCount变量，集合在遍历期间如果内容发生改变，就会改变modCount的值。每次迭代器使用hashNext()/next()遍历下一个元素前，都会检测modCount的值是否是expectedModCount，是的话就遍历，不是就抛出异常
>
>   ​		java.util包下的集合类都是快速失败的
>
> - 安全失败：
>
>   ​		java.util.concurrent包下的容器都是安全失败的，可以在多线程下并发使用
>
>   ​		遍历前把数据复制一份，然后基于复制后的数据进行遍历。遍历时，此时即使原始数据对象被修改了，也不会影响到遍历线程，但是可能导致数据不是最新的

## jdk7和jdk8的区别

1. 实现方式

   jdk7中是数组+链表实现的，jdk8使用的数组+链表+红黑树

2. 新节点插入到链表的顺序不同

   jdk7是头插，jdk8是尾插【防止循环链，导致死锁】

3. jdk8的hash算法优化

   ~~~java
   // jdk7
   final int hash(Object k) {
           int h = hashSeed;
           if (0 != h && k instanceof String) {
               return sun.misc.Hashing.stringHash32((String) k);
           }
   
           h ^= k.hashCode();
   
           // This function ensures that hashCodes that differ only by
           // constant multiples at each bit position have a bounded
           // number of collisions (approximately 8 at default load factor).
           h ^= (h >>> 20) ^ (h >>> 12);
           return h ^ (h >>> 7) ^ (h >>> 4);
       }
   
   //jdk8
    static final int hash(Object key) {
           int h;
           return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
       }
   ~~~

4. 扩容机制优化

   jdk7的条件是size> threshold && null!=table[index];也就是说当前放进来的元素位置被占用了才扩容。否则就直接放  也不扩。

   jdk8 取消了这个限制

5. 节点表示方式

   jdk7中是Entry ，jdk8中是Node  本质是一样的


## hashMap在解决hash冲突方面为什么不用开放型指针，而用链地址法

1. 哈希表容量不能完全利用，并且扩容将会是灾难的，需要删除以前标记过的元素并需要从新计算所有元素的位置，在频繁的删除和插入时效率变得很低。
2. 按上述算法建立起来的哈希表，删除工作非常困难。假如要从哈希表 HT 中删除一个记录，按理应将这个记录所在位置置为空，但我们不能这样做，而只能标上已被删除的标记，否则，将会影响以后的查找。
3. 线性探测法很容易产生堆聚现象。所谓堆聚现象，就是存入哈希表的记录在表中连成一片。按照线性探测法处理冲突，如果生成哈希地址的连续序列愈长(即不同关键字值的哈希地址相邻在一起愈长)，则当新的记录加入该表时，与这个序列发生冲突的可能性愈大。因此，哈希地址的较长连续序列比较短连续序列生长得快，这就意味着，一旦出现堆聚(伴随着冲突)，就将引起进一步的堆聚。

开放型指针，数组中能存的结构是

~~~java
class NodeTyep{
    T key,
    T data
}
~~~

所以每次只需要一直向下找  查看key是否相等即可

## 线程安全ConcurrentHashMap

ConcurrentHashMap是弱一致性的

- **jdk1.7中的数据结构**

  ![1585907039978](E:\文档\学习资料\笔记\面经\HashMap.assets\1585907039978.png)

  - segment数组

    每把锁只锁定容器中的一部分数据，多线程访问容器里不同的数据段的数据，就不会存在锁竞争，提高并发访问率，segment中存储的是map结构的数据，内部拥有一个Entry数组，数组中每个元素又是一个链表。同时segment又继承了ReentrantLock。**HashEntry相对于HashMap中的Entry有一定的差异性：HashEntry中的value以及next都被volatile修饰，这样在多线程读写过程中能够保持它们的可见性，**

  - HashEntry链表

    使用volatile修饰value和next,volatile保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说都是立即可见的，禁止指令重排序，volatile只能保证对单次读写的原子性，i++不能保证原子性，要使用AtomicInt

  - **创建分段锁**

    ​		**JDK7中除了第一个Segment之外，剩余的Segments采用的是延迟初始化的机制：每次put之前都需要检查key对应的Segment是否为null，如果是则调用ensureSegment()以确保对应的Segment被创建。ensureSegment可能在并发环境下被调用，但与想象中不同，ensureSegment并未使用锁来控制竞争，而是使用了Unsafe对象的getObjectVolatile()提供的原子读语义结合CAS来确保Segment创建的原子性。**

    ```java
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))== null) { // recheck
    
    	Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
    
    	while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))== null) {
    
    		if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
    
    			break;
    
    	}
    }
    ```

  - **put/putIfAbsent/putAll**

    ConcurrentHashMap的put方法被代理到了对应的Segment（定位Segment的原理之前已经描述过）中。与JDK6不同的是，**JDK7版本的ConcurrentHashMap在获得Segment锁的过程中，做了一定的优化 - 在真正申请锁之前，put方法会通过tryLock()方法尝试获得锁，在尝试获得锁的过程中会对对应hashcode的链表进行遍历，如果遍历完毕仍然找不到与key相同的HashEntry节点，则为后续的put操作提前创建一个HashEntry。当tryLock一定次数后仍无法获得锁，则通过lock申请锁。**

    需要注意的是，由于在并发环境下，其他线程的put，rehash或者remove操作可能会导致链表头结点的变化，因此在过程中需要进行检查，如果头结点发生变化则重新对表进行遍历。而如果其他线程引起了链表中的某个节点被删除，即使该变化因为是非原子写操作（删除节点后链接后续节点调用的是Unsafe.putOrderedObject()，该方法不提供原子写语义）可能导致当前线程无法观察到，但因为不影响遍历的正确性所以忽略不计。

    之所以在获取锁的过程中对整个链表进行遍历，主要目的是希望遍历的链表被CPU cache所缓存，为后续实际put过程中的链表遍历操作提升性能。

    在获得锁之后，Segment对链表进行遍历，如果某个HashEntry节点具有相同的key，则更新该HashEntry的value值，否则新建一个HashEntry节点，将它设置为链表的新head节点并将原头节点设为新head的下一个节点。新建过程中如果节点总数（含新建的HashEntry）超过threshold，则调用rehash()方法对Segment进行扩容，最后将新建HashEntry写入到数组中。

    put方法中，链接新节点的下一个节点（HashEntry.setNext()）以及将链表写入到数组中（setEntryAt()）都是通过Unsafe的putOrderedObject()方法来实现，这里并未使用具有原子写语义的putObjectVolatile()的原因是：JMM会保证获得锁到释放锁之间所有对象的状态更新都会在锁被释放之后更新到主存，从而保证这些变更对其他线程是可见的。

  - **get/containsKey**

    他们都没有使用锁，而是通过Unsafe对象的getObjectVolatile()方法提供的原子读语义，来获得Segment以及对应的链表，然后对链表遍历判断是否存在key相同的节点以及获得该节点的value。**但由于遍历过程中其他线程可能对链表结构做了调整，因此get和containsKey返回的可能是过时的数据，这一点是ConcurrentHashMap在弱一致性上的体现。**如果要求强一致性，那么必须使用Collections.synchronizedMap()方法。

  - **size/contiansValue**

    ​		首先不加锁循环执行以下操作：循环所有的Segment（通过Unsafe的getObjectVolatile()以保证原子读语义），获得对应的值以及所有Segment的modcount之和。如果连续两次所有Segment的modcount和相等，则过程中没有发生其他线程修改ConcurrentHashMap的情况，返回获得的值。

    当循环次数超过预定义的值时，这时需要对所有的Segment依次进行加锁，获取返回值后再依次解锁。值得注意的是，加锁过程中要强制创建所有的Segment，否则容易出现其他线程创建Segment并进行put，remove等操作：

  - 注意

    一般来说，应该避免在多线程环境下使用size和containsValue方法。

    注1：modcount在put, replace, remove以及clear等方法中都会被修改。

    注2：对于containsValue方法来说，如果在循环过程中发现匹配value的HashEntry，则直接返回true。

    最后，与HashMap不同的是，ConcurrentHashMap并不允许key或者value为null，按照Doug Lea的说法，这么设计的原因是在ConcurrentHashMap中，一旦value出现null，则代表HashEntry的key/value没有映射完成就被其他线程所见，需要特殊处理。在JDK6中，get方法的实现中就有一段对HashEntry.value == null的防御性判断。但Doug Lea也承认实际运行过程中，这种情况似乎不可能发生

  - 高并发

  分段锁技术，segment继承于ReentrantLock

  HashEntry<K,V> node = tryLock() ? null :

  ​                scanAndLockForPut(key, hash, value);

  尝试获取锁，失败说明有其他线程竞争，则利用scanAndLockForPut()自旋获取锁

  - CAS乐观锁

  - 在读取数据的时候不进行加锁，在准备会写的时候，比较原值是否被修改过，如果被修改过，则重新执行读取流程

  - ABA问题

    - 一个线程把值改成了B，又来一个线程改回了A，此时再来的线程不知道这个值被改过
    - 通过版本号方式，或者是时间戳方式

  - **jdk1.7中的并发度问题**

    ​		并发度可以理解为程序运行时能够同时更新ConccurentHashMap且不产生锁竞争的最大线程数，实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度。ConcurrentHashMap默认的并发度为16，但用户也可以在构造函数中设置并发度。当用户设置并发度时，ConcurrentHashMap会使用大于等于该值的最小2幂指数作为实际并发度（假如用户设置并发度为17，实际并发度则为32）。运行时通过将key的高n位（n = 32 – segmentShift）和并发度减1（segmentMask）做位与运算定位到所在的Segment。segmentShift与segmentMask都是在构造过程中根据concurrency level被相应的计算出来。

    ​		如果并发度设置的过小，会带来严重的锁竞争问题；如果并发度设置的过大，原本位于同一个Segment内的访问会扩散到不同的Segment中，CPU cache命中率会下降，从而引起程序性能下降。

- **jdk8的数据结构**

  - 1.8中放弃了`Segment`臃肿的设计，取而代之的是采用`Node` + `CAS` + `Synchronized+红黑树 `来保证并发安全进行实现，结构如下：

    ![1585907052503](E:\文档\学习资料\笔记\面经\HashMap.assets\1585907052503.png)

  - **sizeCtl**

    它是一个控制标识符，在不同的地方有不同用途，而且它的取值不同，也代表不同的含义。

    - 负数代表正在进行初始化或扩容操作
- -1代表正在初始化
  
    - -N 表示有N-1个线程正在进行扩容操作
- 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，这一点类似于扩容阈值的概念。**它的值始终是当前ConcurrentHashMap容量的0.75倍，**这与loadfactor是对应的。
  
- **重要的类**
  
  - Node
  
    key-value键值对，并且将key和value设置为volatile同步锁
  
  - TreeNode
  
    树节点类，当链表长度过长的时候，会转换为TreeNode。**它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中**
  
    - **TreeBin**

      **这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。另外这个类还带有了读写锁。**
  
  - ### 扩容方法 transfer
  
    它支持多线程进行扩容操作，**而并没有加锁**。我想这样做的目的不仅仅是为了满足concurrent的要求，而是希望利用并发处理去减少扩容带来的时间影响。因为在扩容的时候，总是会涉及到从一个“数组”到另一个“数组”拷贝的操作，如果这个操作能够并发进行，那真真是极好的了。
  
    整个扩容操作分为两个部分
  
    -  第一部分是构建一个nextTable,它的容量是原来的两倍，这个操作是单线程完成的。这个单线程的保证是通过RESIZE_STAMP_SHIFT这个常量经过一次运算来保证的，这个地方在后面会有提到；
    - 第二个部分就是将原来table中的元素复制到nextTable中，这里允许多线程进行操作。
    
    先来看一下单线程是如何完成的：
    
    它的大体思想就是遍历、复制的过程。首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素：
    
    - 如果这个位置为空，就在原table中的i位置放入forwardNode节点，这个也是触发并发扩容的关键点；
    
    - 如果这个位置是Node节点（fh>=0），如果它是一个链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上
    
    - 如果这个位置是TreeBin节点（fh<0），也做一个反序处理，并且判断是否需要untreefi，把处理的结果分别放在nextTable的i和i+n的位置上
    
    - 遍历过所有的节点以后就完成了复制工作，这时让nextTable作为新的table，并且更新sizeCtl为新容量的0.75倍 ，完成扩容。
    
    再看一下多线程是如何完成的：
    
    在代码的69行有一个判断，如果遍历到的节点是forward节点，就向后继续遍历，再加上给节点上锁的机制，就完成了多线程的控制。多线程遍历节点，处理了一个节点，就把对应点的值set为forward，另一个线程看到forward，就向后遍历。这样交叉就完成了复制工作。而且还很好的解决了线程安全的问题。 
  
- JDK6,7中的ConcurrentHashmap主要使用Segment来实现减小锁粒度，把HashMap分割成若干个Segment，在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。

  jdk7中ConcurrentHashmap中，当长度过长碰撞会很频繁，链表的增改删查操作都会消耗很长的时间，影响性能,所以jdk8 中完全重写了concurrentHashmap,代码量从原来的1000多行变成了 6000多 行，实现上也和原来的分段式存储有很大的区别。

  主要设计上的变化有以下几点: 

  1. 不采用segment而采用node，锁住node来实现减小锁粒度。
  2. 设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。
  3. 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。
  4. sizeCtl的不同值来代表不同含义，起到了控制的作用。

## ArrayList

- 底层结构

  Object []elementData

- 构造函数

  通过无参构造方法的方式ArrayList()初始化，则赋值底层数Object[] elementData为一个默认空数组Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}所以数组容量为0，**只有真正对数据进行添加add时，才分配默认DEFAULT_CAPACITY = 10的初始容量。**

- 扩容机制

  ArrayCount = ArrayCount  + ArrayCount /2

- **ArrayList（int initialCapacity）会不会初始化数组大小？**

  **会初始化数组大小！但是List的大小没有变，因为list的大小是返回size的。**

  而且将构造函数与initialCapacity结合使用，然后使用set（）会抛出异常，尽管该数组已创建，但是大小设置不正确。

  使用sureCapacity（）也不起作用，因为它基于elementData数组而不是大小。

  还有其他副作用，这是因为带有sureCapacity（）的静态DEFAULT_CAPACITY。

  进行此工作的唯一方法是在使用构造函数后，根据需要使用add（）多次。
  
- JDK7和JDK8的区别

  - 1.7

    1.7以前会调用this(10)才是真正的容量为10，1.7即本身以后是默认走了空数组，只有第一次add的时候容量会变成10。