# JVM探索

- 请你谈谈你对JVM的理解？java8虚拟机和之前的变化更新？
- 什么是OOM，什么是栈溢出StackOverFlowError？怎么分析？
- JVM的常用调优参数有哪些？
- 内存快照如何抓取，怎么分析Dump文件？知道吗？
- 谈谈JVM中，类加载器你的认识？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204092559109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204092529560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)



1. JVM的位置

   - java程序跑在jvm上面，jre-jvm， 在操作系统之上，最底下是硬件系统。

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204091904450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

2. JVM的体系结构

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817162604620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

   - [体系结构](https://www.processon.com/view/5ea567b163768974669293f3)
   - [jvm内存模型](https://www.processon.com/view/5c31d6e2e4b0fa03ce8d3017)
   - jvm调优说的是“方法区”和“堆“，而”栈“、”本地方法栈“、”程序计数器“不存在垃圾回收一事
   - 虚拟机试图使用最大内存为电脑内存的1/4, 而jvm初始化内存为1/64（**-Xms1024m -Xmx1024m -XX:+PrintGCDetails**）

3. 类加载器

   - 定义： 类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个 java.lang.Class对象，用来封装类在方法区内的数据结构

   - 作用：加载类文件，引用在栈中，具体实例在堆中
   - 虚拟器自带三种类加载器
     - 启动类（根）加载器
     - 扩展类加载器
     - 应用程序加载器（系统类加载器）

4. 双亲委派机制

   - [Java类加载机制，你理解了吗](https://baijiahao.baidu.com/s?id=1636309817155065432&wfr=spider&for=pc)
   - [面试官：java双亲委派机制及作用](https://www.jianshu.com/p/1e4011617650)
   - 当某个类加载器需要加载某个`.class`文件时，它首先把这个任务委托给他的上级类加载器，递归这个操作，如果上级的类加载器没有加载，自己才会去加载这个类。

   ```java
   package com.xiaofan;
   
   public class Car {
       public static void main(String[] args) {
           Car car1 = new Car();
           Car car2 = new Car();
           Car car3 = new Car();
   
           System.out.println(car1.hashCode());
           System.out.println(car2.hashCode());
           System.out.println(car3.hashCode());
   
           Class<? extends Car> aClass = car1.getClass();
   
           System.out.println(aClass.getClassLoader());    // 应用程序加载器 AppClassLoader
           System.out.println(aClass.getClassLoader().getParent()); // 扩展类加载器 ExtClassLoader       D:\jdk1.8\jre\lib\ext/*.jar
           System.out.println(aClass.getClassLoader().getParent().getParent());  // null   1. 不存在， 2. java程序获取不到  D:\jdk1.8\jre\lib\rt.jar
   
       }
   }
   ```

   

5. 沙箱安全机制

   - Java安全模型核心
   - 沙箱：限制程序的运行时环境
   - 域Domain概念
   - 将java代码限定在虚拟机特定的运行范围内，严格限制代码对本地系统资源的访问，这样措施保证对代码的有效隔离，防止对本地系统的破坏

6. Native

   - 调用底层c，c++语言库
   - native -> jni -> 本地方法接口 -> 本地方法库
   - 本地方法栈
   - 一般用的不多，硬件开发用的多

7. PC程序计数器

   - pc寄存器
   - 程序计数器Program Counter Register
   - 每个线程都有一个程序计数器，线程私有，就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址， 也即即将要执行的指令代码）
   - 非常狭小的空间 --可以忽略不计

8. 方法区

   - 所有线程共享
   - 静态变量，常量，类信息（构造方法，接口定义）运行时的常量池也存在方法区中，但是实例变量存在堆内存中，和方法区无关

9. 栈

   - 栈：先进后出，后进先出，堆：先进先出，后进后出
   - 线程结束，栈内存释放
   - 主管程序的运行
   - 存放基本类型，实例方法，对象引用...
   - 栈帧：父帧子帧，每一个在执行的方法都会产生栈帧
   - 栈满报错：StackOverflowError

10. 三种JVM

    - SUN 公司 - HotSpot （使我们主要研究的）
    - BEA 公司 JRocket
    - IBM 公司 J9

11. 堆

    - 一个JVM只有一个堆内存

    - 保存我们所有引用类型的真实对象

    - 3个区域

      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204104907578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

      - 新生区（伊甸园）: 所有的对象都new在了伊甸园去，从中出生
      - 养老区-- 老年区 --从新生区（幸存0幸存1）中顺下来的，干不掉，杀不死
      - 永久区
        - 此区域不存在垃圾回收，但是也会崩（ 启动了大量第三方jar） -- 永久区是一个常驻内存区域，用于存放JDK自身携带的Class Interface的元数据
        - jdk1.6之前永久代，常量池是在`方法区`
        - jdk1.7永久代，慢慢退化，去永久代，常量池在`堆`中 
          - jdk1.8无永久代，常量池在`元空间`
        - 关闭虚拟机就会释放这块内存

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204110517104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020411071448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

    

    - gc垃圾回收主要针对新生区（伊甸园区）和养老区

      ```java
      public class Code {
          public static void main(String[] args) {
              long max = Runtime.getRuntime().maxMemory();
      
              long init = Runtime.getRuntime().totalMemory();
      
              System.out.println("max=" + max + "字节\t" + (max/(double)1024/1024) + "MB");
              System.out.println("init=" + init + "字节\t" + (init/(double)1024/1024) + "MB");
      
              // 默认情况下：分配的总内存（max）为电脑内存的1/4,初始化内存(init)为电脑内存的1/64
          }
      }
      ```

      ```shell
      max:981.5M
      init:981.5M
      Heap
       PSYoungGen      total 305664K, used 15729K [0x00000000eab00000, 0x0000000100000000, 0x0000000100000000)
        eden space 262144K, 6% used [0x00000000eab00000,0x00000000eba5c420,0x00000000fab00000)
        from space 43520K, 0% used [0x00000000fd580000,0x00000000fd580000,0x0000000100000000)
        to   space 43520K, 0% used [0x00000000fab00000,0x00000000fab00000,0x00000000fd580000)
       ParOldGen       total 699392K, used 0K [0x00000000c0000000, 0x00000000eab00000, 0x00000000eab00000)
        object space 699392K, 0% used [0x00000000c0000000,0x00000000c0000000,0x00000000eab00000)
       Metaspace       used 2998K, capacity 4496K, committed 4864K, reserved 1056768K
        class space    used 322K, capacity 388K, committed 512K, reserved 1048576K
      
      ```

    - **抛出OOM demo演示**

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204132553854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204132821475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

    本地下载地址：http://www.downcc.com/soft/290199.html

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204140443665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

    ```java
    package com.xiaofan;
    
    import java.util.Random;
    
    public class Main {
    
        public static void main(String[] args) {
            String a = "kuangshensayjava";
            while(true) {
                a += a + new Random().nextInt(999999999) + new Random().nextInt(999999999);
            }
        }
    }
    
    ```

    ```java
    D:\jdk1.8\bin\java.exe -Xms8m -Xmx8m -XX:+PrintGCDetails -javaagent:E:\idea_ultimate\IntelliJ_IDEA_2020_2\lib\idea_rt.jar=50684:E:\idea_ultimate\IntelliJ_IDEA_2020_2\bin -Dfile.encoding=UTF-8 -classpath D:\jdk1.8\jre\lib\charsets.jar;D:\jdk1.8\jre\lib\deploy.jar;D:\jdk1.8\jre\lib\ext\access-bridge-64.jar;D:\jdk1.8\jre\lib\ext\cldrdata.jar;D:\jdk1.8\jre\lib\ext\dnsns.jar;D:\jdk1.8\jre\lib\ext\jaccess.jar;D:\jdk1.8\jre\lib\ext\jfxrt.jar;D:\jdk1.8\jre\lib\ext\localedata.jar;D:\jdk1.8\jre\lib\ext\nashorn.jar;D:\jdk1.8\jre\lib\ext\sunec.jar;D:\jdk1.8\jre\lib\ext\sunjce_provider.jar;D:\jdk1.8\jre\lib\ext\sunmscapi.jar;D:\jdk1.8\jre\lib\ext\sunpkcs11.jar;D:\jdk1.8\jre\lib\ext\zipfs.jar;D:\jdk1.8\jre\lib\javaws.jar;D:\jdk1.8\jre\lib\jce.jar;D:\jdk1.8\jre\lib\jfr.jar;D:\jdk1.8\jre\lib\jfxswt.jar;D:\jdk1.8\jre\lib\jsse.jar;D:\jdk1.8\jre\lib\management-agent.jar;D:\jdk1.8\jre\lib\plugin.jar;D:\jdk1.8\jre\lib\resources.jar;D:\jdk1.8\jre\lib\rt.jar;E:\idea_ultimate\Project\Hello\out\production\Hello com.xiaofan.Main
    [GC (Allocation Failure) [PSYoungGen: 1520K->504K(2048K)] 1520K->632K(7680K), 0.0008567 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 1973K->488K(2048K)] 2101K->1314K(7680K), 0.0006303 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 1578K->488K(2048K)] 2404K->1619K(7680K), 0.0004381 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 1576K->224K(2048K)] 3766K->2944K(7680K), 0.0006442 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 782K->224K(2048K)] 4562K->4004K(7680K), 0.0004239 secs] [Times: user=0.03 sys=0.02, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 224K->224K(2048K)] 4004K->4004K(7680K), 0.0004558 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC (Allocation Failure) [PSYoungGen: 224K->0K(2048K)] [ParOldGen: 3780K->2165K(5632K)] 4004K->2165K(7680K), [Metaspace: 2947K->2947K(1056768K)], 0.0064615 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
    [Full GC (Ergonomics) [PSYoungGen: 1134K->0K(2048K)] [ParOldGen: 5345K->2697K(5632K)] 6480K->2697K(7680K), [Metaspace: 2963K->2963K(1056768K)], 0.0057346 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
    [GC (Allocation Failure) [PSYoungGen: 39K->160K(2048K)] 4856K->4976K(7680K), 0.0003290 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 160K->160K(2048K)] 4976K->4976K(7680K), 0.0002194 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC (Allocation Failure) [PSYoungGen: 160K->0K(2048K)] [ParOldGen: 4816K->3760K(5632K)] 4976K->3760K(7680K), [Metaspace: 2966K->2966K(1056768K)], 0.0020017 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] 3760K->3760K(7680K), 0.0002045 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] [ParOldGen: 3760K->3741K(5632K)] 3760K->3741K(7680K), [Metaspace: 2966K->2966K(1056768K)], 0.0064159 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
    Heap
     PSYoungGen      total 2048K, used 131K [0x00000000ffd80000, 0x0000000100000000, 0x0000000100000000)
      eden space 1536K, 8% used [0x00000000ffd80000,0x00000000ffda0c38,0x00000000fff00000)
      from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
      to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
     ParOldGen       total 5632K, used 3741K [0x00000000ff800000, 0x00000000ffd80000, 0x00000000ffd80000)
      object space 5632K, 66% used [0x00000000ff800000,0x00000000ffba7700,0x00000000ffd80000)
     Metaspace       used 3075K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 332K, capacity 388K, committed 512K, reserved 1048576K
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
    	at java.util.Arrays.copyOf(Arrays.java:3332)
    	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
    	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
    	at java.lang.StringBuilder.append(StringBuilder.java:208)
    	at com.xiaofan.Main.main(Main.java:10)
    ```

    - JProfiler工具分析OOM的原因

    `-Xms 设置初始化内存大小`

    `-Xmx 设置最大分配内存`

    `-XX:+HeapDumpOnOutOfMemoryError `

    `-XX:+PrintGCDetails`

    ```java
    package com.xiaofan;
    
    import java.util.ArrayList;
    
    public class Main {
    
        byte [] arr = new byte[1024*1024]; //1m
    
        public static void main(String[] args) {
            ArrayList<Main> list = new ArrayList<>();
            int count = 0;
            try {
                while(true) {
                    list.add(new Main());
                    count += 1;
                }
            } catch (Exception e) {
                System.out.println("count: " + count);
                e.printStackTrace();
            }
    
        }
    }
    ```

12. 堆内存调优

    - 报OOM时，首次按尝试扩大堆内存空间查看结果，分析内存，查看一下哪个地方出现问题（JProfiler）
    - JProfiler作用：分析DumpN内存文件，快速定位内存泄漏，获得堆中的数据，获取大的对象。
    - 虚拟机基本配置参数
      - -Xms 设置Java程序启动时的初始堆大小-- 初始化内存大小
      - -Xmx 设置java程序能获得的最大堆大小-- 最大内存大小
      - -XX:+HeapDumpOnOutOfMemoryError 使用改参数可以在内存溢出时导出这个堆信息
      - -XX:+HeapDumpPath， 可以设置导出堆的存放路径
      - -XX:+PrintGCDetails 打印GC垃圾回收信息
      - 举例 ----- -Xms1m -Xmx1m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/Test3.dump

13. GC

    - 垃圾回收为自动，手动只能提醒
    - GC作用于堆+方法区
    - GC大部分针对新生代
      - 轻GC ----- 普通GC
      - 重GC ----- 全局GC
    - GC算法
      - 复制算法 ---[GC算法-复制算法](https://www.cnblogs.com/hujingnb/p/12642079.html)
        - 该算法将内存平均分成两部分，然后每次只使用其中的一部分，当这部分内存满的时候，将内存中所有存活的对象复制到另一个内存中，然后将之前的内存清空，只使用这部分内存，循环下去
          - 幸存区01， from...to...， 0和1互相不断交换，进行gc进行复制算法
          - 若一直没有死进入到养老区
            - 优点：实现简单，不产生内存碎片
            - 缺点：浪费一半的内存空间
    - 标记清除算法 -----扫描对象，对活着的对象进行标记， 对没有标记的对象进行清除
      - 优点：不需要额外空间，优于复制算法
      - 两次扫描，浪费时间，会存在内存碎片
    - 标记压缩算法 ----- 再优化，压缩：防止碎片的产生， 方法： 向一端移动活的对象，多了一个移动成本
    - 标记清除压缩算法 ----- 先标记清除几次再进行压缩，等碎片多了之后
    - 引用计数算法 ------ 每个对象一个计数器，一般不用，因为计数器有消耗，用过多次的不删，0次的就删除了 ---引用出现+1,引用删除-1
    - 总结：
      - 内存效率：时间复杂度：复制算法 > 标记清除 >标记压缩
      - 内存整齐度：复制算法=标记压缩>标记清除
      - 内存利用率 ----- 标记压缩=标记清除 > 复制算法
      - 分代收集算法（JVM调优）： 没有最好的算法，只有最合适的
        - 年轻代 ----- 存活率低，复制算法
        - 老年代 ----- 存活率高， 标记清除与标记压缩混合实现

14. JMM

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200818164234102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- Java Memory Model: Java内存模型 -- 抽象概念， 理论
- 定义了线程工作内存和主内存之间的抽象关系
  - 线程之间的共享变量存储在主内存中 ----MAIN MEMORY
  - 每个线程都有一个私有的本地内存 ---- LOCAL MEMORY
- 与并发相关



16. OOM的种类和原因

- java.lang.OutOfMemoryError
- 内存泄漏（memory leak） 是指程序中一动态分配的堆内存由于某种原因程序未释放，造成系统内存的浪费，导致程序运行减慢甚至系统奔溃等严重后果
- 内存溢出（out of memory）是指程序申请内存时， 没有足够的内存供申请者使用，说白就是内存不够用，此时就会报错OOM,即所谓的内存溢出
- OOM种类和原因
  - java堆溢出 ---- 既然堆是存放实例对象的，那就无限创建实例对象
  - 虚拟机栈溢出 ----- 虚拟机栈描述的是java方法执行的内存模型， 每个方法在执行的时候都会创建一个栈帧用于存储局部变量表，操作数栈、动态链接、方法出口等信息，本地方法栈和虚拟栈的区别是，虚拟机栈为虚拟机运行java方法服务，而本地方法栈为虚拟机提供native方法服务；在单线程的操作中，无论是由于栈帧太大，还是虚拟栈空间太小，当占空间无法分配时，虚拟机抛出的都是StackOverflowError，而不会得到OutOfMemoryError异常，在多线程情况下，则会抛出OutOfMemoryError异常
  - 本地方法栈溢出！
  - 方法区和运行时常量池溢出 - java堆和方法区： java堆区主要存放对象实例和数组等，方法区保存类信息，静态变量等等，运行时常量也是方法区的一部分， 这两块区域是线程共享的区域，只会抛出OutOfMemeryError。
  - 本机内存直接溢出 -- NIO有关（New input/Output）,引入了一种基于通道与缓存区的I/O方式，可是native函数库直接分配堆外内存， 然后通过一个存储在java堆中的对象作为这块内存的应用进行操作。这样能在一些场景中显著提高性能，因为避免了再Java堆和Native堆中来回复制数据

	1. 百度

 	2. 思维导图