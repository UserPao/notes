# Linux下cpu过高的问题

## 第一种

### 1. 使用top命令定位异常进程。

使用top命令定位异常进程。可以看见12836的CPU和内存占用率都非常高

![img](E:\文档\学习资料\笔记\面经\Linux下的问题.assets\20160503111524410.png)

此时可以再执行ps -ef | grep java，查看所有的java进程，在结果中找到进程号为12836的进程，即可查看是哪个应用占用的该进程。

### 2. 使用top -H -p 进程号查看异常线程

![img](https://img-blog.csdn.net/20160503111715099?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 3. 使用printf "%x\n" 线程号将异常线程号转化为16进制

![img](https://img-blog.csdn.net/20160503112248412?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

或者是echo 'obase=16;12875' | bc

bc是linux的计算器命令

### 4. 使用jstack 进程号|grep 16进制异常线程号 -A90来定位异常代码的位置

使用jstack 进程号|grep 16进制异常线程号 -A90来定位异常代码的位置（最后的-A90是日志行数，也可以输出为文本文件或 、使用其他数字）。可以看到异常代码的位置。

![img](https://img-blog.csdn.net/20160503112227975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息。 Jstack工具可以用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

## 第二种

### 1.使用top指令，然后按shift + p按照CPU排序

找到占用CPU过高的进程

### 2. 使用ps -mp pid -o THREAD,tid,time | sort -rn 

获取线程信息，并找到占用CPU过高的线程

### 3. 使用echo 'obase=16;[线程id]' | bc或者printf "%x\n" [线程id] 

将需要的线程ID转换成16进制格式

### 4. 使用jstack pid |grep tid -A 30 [线程id的16进制] 

打印出来线程的堆栈信息

# 用户态和内核态

- 内核态

  ​		cpu可以访问内存的所有数据，包括外围设备，例如硬盘，网卡，cpu也可以将自己从一个程序切换到另一个程序。

- 用户态

  ​		只能受限的访问内存，且不允许访问外围设备，占用cpu的能力被剥夺，cpu资源可以被其他程序获取。

- 为什么要有用户态和内核态？

  ​		由于需要限制不同的程序之间的访问能力, 防止他们获取别的程序的内存数据, 或者获取外围设备的数据, 并发送到网络, CPU划分出两个权限等级 -- 用户态和内核态。

- 用户态和内核态的切换

  ​		所有用户程序都是运行在用户态的, 但是有时候程序确实需要做一些内核态的事情, 例如从硬盘读取数据, 或者从键盘获取输入等. 而唯一可以做这些事情的就是操作系统, 所以此时程序就需要先操作系统请求以程序的名义来执行这些操作

  ​		这个时候就需要一个机制：用户态程序切换到内核态，但是不能控制在内核态中执行的指令，这种机制叫系统调用，在cpu中称之为陷阱指令。

  ​		他的工作流程：

     1. 用户态程序将一些数据值存放在寄存器中，或者使用参数创建一个堆栈，以此表明需要操作系统提供的服务

     2. 用户态程序执行陷阱指令

     3. cpu切换到内核态，并跳到位于内存指定位置的指令，这些指令是操作系统的一部分，他们具有内存保护，不可被用户态程序访问

     4. 这些指令称之为陷阱，或者系统调用处理器，他们会读取程序放入内存的数据参数，并执行程序请求的服务

     5. 系统调用完成后，操作系统会重置cpu为用户态并返回系统调用的结果

        **用户态切换到内核态的三种方式**

        1. 系统调用

           ​		这是用户态进程主动要求切换到内核态的一种方式，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作，比如前例中fork()实际上就是执行了一个创建新进程的系统调用。而系统调用的机制其核心还是使用了操作系统为用户特别开放的一个中断来实现。

        2. 异常

           ​		当CPU在执行运行在用户态下的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中，也就转到了内核态，比如缺页异常。

        3. 外围设备的中断

           ​		当外围设备完成用户请求的操作后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条即将要执行的指令转而去执行与中断信号对应的处理程序，如果先前执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了由用户态到内核态的切换。比如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。

        **涉及到由用户态切换到内核态的步骤主要包括：**

        [1] 从当前进程的描述符中提取其内核栈的ss0及esp0信息。

        [2] 使用ss0和esp0指向的内核栈将当前进程的cs,eip,eflags,ss,esp信息保存起来，这个

        过程也完成了由用户栈到内核栈的切换过程，同时保存了被暂停执行的程序的下一

        条指令。

        [3] 将先前由中断向量检索得到的中断处理程序的cs,eip信息装入相应的寄存器，开始

        执行中断处理程序，这时就转到了内核态的程序执行了。


# 常用命令

1. **cd**

   change directory

   ~~~shell
   cd /system/bin # 表示切换到/system/bin路径下。
   cd logs # 表示切换到logs路径下。
   cd / # 表示切换到根目录。
   cd ../ # 表示切换到上一层路径。
   ~~~

2. **ls**

   list

   ~~~shell
   ls / # 显示根目录下的所有文件及文件夹。
   ls -l /data # 显示/data路径下的所有文件及文件夹的详细信息。
   ls -l # 显示当前路径下的所有文件及文件夹的详细信息
   ls *l wc #显示当前目录下面的文件数量。
   ~~~

3. **cat**

   concatenate 读取文件内容及拼接文件。

   ~~~shell
   cat /sys/devices/system/cpu/online # 读取 /sys/devices/system/cpu/路径下online文件内容。
   cat test.txt # 读取当前路径下test.txt文件内容。
   ~~~

4. **rm**

   remove 

   ~~~shell
   # 常用参数-r -f，-r表示删除目录，也可以用于删除文件，-f表示强制删除，不需要确认
   rm -rf path # 删除path。
   rm test.txt # 删除test.txt。
   ~~~

5. **mkdir**

   make directory  创建文件夹

   ~~~shell
   mkdir /data/path # 在/data路径下创建path文件夹。
   mkdir -p a/b/c # 参数 -p用于创建多级文件夹，这句命令表示在当前路径下创建文件夹a， 而a文件夹包含子文件夹b，b文件夹下又包含子文件夹c。
   ~~~

6. **cp**

   copy  复制文件或文件夹。

   ~~~shell
   cp /data/logs /data/local/tmp/logs # 复制/data路径下的logs到/data/local/tmp路径下。
   cp 1.sh /sdcard/ # 复制当前路径下的1.sh到/sdcard下。
   ~~~

7. **kill**

   kill PID码

   ~~~shell
   ps au  # 查看进程。找到要终止的进程的PID
   kill 3163 # 终止进程ID为3163的进程
   kii -9 3163 # 强制终止进程ID为3163的进程
   ~~~

8. **shell脚本文件之"hello world"**

   ~~~shell
   #!/bin/sh
   a="hello world!"
   
   num=2
   
   echo "a is : $a num is : ${num}nd"
   
   # 运行结果 a is : hello world! num is : 2nd
   ~~~

9. **shell脚本文件之遍历目录**

   ~~~shell
   问题：
   1. 切换工作目录至/tmp
   2. 依次向/tmp目录中的每个文件或子目录问好（Hello,log）
   3. 统计/tmp目录下共有多个文件，并显示出来
   
   #!/bin/bash
   cd /tmp
   for i in /tmp/*
   do
       echo "Hello , $i"
   done
   count=`ls -l|grep '^-'|wc -l`
   echo "====file_count:$count===="
   ~~~

10. **vim**

    ```shell
    vim a.txt  # 编辑a.txt文件
    ```
    
    几种命令格式
    
    - **i 切换到输入模式，以输入字符。**
    - **x 删除当前光标所在处的字符。**
    - **: 切换到底线命令模式，以在最底一行输入命令**
    
    ![img](E:\文档\学习资料\笔记\面经\Linux下的问题.assets\2019010522374780.png)
    
    ![img](E:\文档\学习资料\笔记\面经\Linux下的问题.assets\20190105223929889.png)
    
11. **Jstack**

      		jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息。 Jstack工具可以用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

12. **pwd**

    查看当前路径

13. 



