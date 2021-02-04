# 1. 什么是JUC

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917184122493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

# 2. 进程和线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917184730638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```java
// 本地方法，底层的C++，java无法直接操作硬件
private native void start0();
```

- 查看CPU核数的方法
  - 任务管理器 - 性能 - CPU
  - 我的电脑-右键管理-设备管理器-处理器

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091719015659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



- 线程有几个状态

```java
public enum State {
       	// 新生
        NEW,
		// 运行
        RUNNABLE,
		// 阻塞
        BLOCKED,
		// 等待
        WAITING,
		// 超时等待
        TIMED_WAITING,
		// 终止
        TERMINATED;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091719120031.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



# 3. Lock锁

- idea 快捷键 `Ctrl + Alt +T`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918084701655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918085321432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

公平锁：先来的先执行

**非公平锁：十分不公平，可以插队（默认）**

- Synchronized

```java
package com.xiaofan;

public class SaleTicketDemo01 {
    public static void main(String[] args) {
        // 并发： 多线程操作同一个资源类， 把资源类丢入线程
        final Ticket ticket = new Ticket();

        // @FunctionalInterface 函数式接口, jdk1.8 lambda 表达式(参数)->{代码}
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket.sale(); }, "A").start();
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket.sale(); }, "B").start();
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket.sale(); }, "C").start();
    }
}

// 资源类 OOP
class Ticket {
    // 属性，方法
    private int number = 30;

    // 卖票的方式
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "张票, 剩余票数：" + number);
        }
    }
}

```

- Lock

```java
package com.xiaofan;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class SaleTicketDemo02 {
    public static void main(String[] args) {
        // 并发： 多线程操作同一个资源类， 把资源类丢入线程
        final Ticket2 ticket2 = new Ticket2();

        // @FunctionalInterface 函数式接口, jdk1.8 lambda 表达式(参数)->{代码}
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket2.sale(); }, "A").start();
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket2.sale(); }, "B").start();
        new Thread(()->{ for (int i = 0; i < 31; i++) ticket2.sale(); }, "C").start();

    }
}

// Lock三部曲
// 1. new ReentrantLock();
// 2. lock.lock();
// 3. lock.unlock(); 解锁
class Ticket2 {
    // 属性，方法
    private int number = 30;

    private Lock lock = new ReentrantLock();

    // 卖票的方式
    public void sale() {

        lock.lock();

        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "张票, 剩余票数：" + number);
            }
        }catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

- Synchronized 和 Lock的区别
  1. Synchronized 内置的Java关键字， Lock是一个Java类
  2. Synchronized 无法判断获取锁的状态， Lock可以判断是否获取到了锁
  3. Synchronized 会自动释放锁， Lock必须要手动释放锁，如果不释放锁，死锁！
  4. Synchronized 线程1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下去
  5. Synchronized 可重入锁，不可以中断的，非公平锁；Lock，可重入锁，可以判断锁，非公平锁（可自己设置）
  6. Synchronized 适合锁少量的代码同步问题， Lock适合锁大量的同步代码！



# 4. 生产者和消费者

![在这里插入图片描述](https://img-blog.csdnimg.cn/202009181016296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## 4.1. 生产者和消费者问题Synchronized版

***面试手写的： 单例模式、排序算法、生产者和消费者、死锁***

```java
package com.xiaofan.pc;


/**
 * 线程之间的通信问题： 生产者和消费者问题！ 等待环形， 通知唤醒
 * 线程交替执行   A B 操作同一个变量 num = 0
 * A num + 1
 * B num - 1
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.decrement();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}

// 判断等待， 业务， 通知
class Data {

    private int number = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            this.wait();
        }
        number ++;
        System.out.println(Thread.currentThread().getName() + " => " + number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }

    //-1
    public synchronized void decrement() throws InterruptedException {
        if (number == 0) {
            this.wait();
        }
        number --;
        System.out.println(Thread.currentThread().getName() + " => " + number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }
}
```



- 问题存在，A,B,C,D 四个线程！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918094928811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```java
package com.xiaofan.pc;


/**
 * 线程之间的通信问题： 生产者和消费者问题！ 等待环形， 通知唤醒
 * 线程交替执行   A B 操作同一个变量 num = 0
 * A num + 1
 * B num - 1
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.decrement();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.decrement();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "C").start();

        new Thread(()->{
            try {
                for (int i = 0; i < 10; i++) data.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "D").start();
    }
}

// 判断等待， 业务， 通知
class Data {

    private int number = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        while (number != 0) {
            this.wait();
        }
        number ++;
        System.out.println(Thread.currentThread().getName() + " => " + number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }

    //-1
    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            this.wait();
        }
        number --;
        System.out.println(Thread.currentThread().getName() + " => " + number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }
}
```

## 4.2. JUC版本的生产者和消费者

```java
package com.xiaofan.pc;


import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 线程之间的通信问题： 生产者和消费者问题！ 等待环形， 通知唤醒
 * 线程交替执行   A B 操作同一个变量 num = 0
 * A num + 1
 * B num - 1
 */
public class B {
    public static void main(String[] args) {
        Data1 data1 = new Data1();
        new Thread(()->{ for (int i = 0; i < 10; i++) data1.increment(); }, "A").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) data1.decrement(); }, "B").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) data1.increment(); }, "C").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) data1.decrement(); }, "D").start();

    }
}

// 判断等待， 业务， 通知
class Data1 {

    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    //+1
    public void increment() {
        lock.lock();

        try{
            while (number != 0) {
                condition.await();
            }
            number ++;
            System.out.println(Thread.currentThread().getName() + " => " + number);
            // 通知其他线程， 我+1完毕了
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    //-1
    public void decrement() {
        lock.lock();

        try {
            while (number == 0) {
                condition.await();
            }
            number --;
            System.out.println(Thread.currentThread().getName() + " => " + number);
            // 通知其他线程， 我+1完毕了
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }


    }
}

```



## 4.3.  Condition实现精准通知唤醒

```java
package com.xiaofan.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 多线程轮流打印 A-B-C
 */
public class C {
    public static void main(String[] args) {
        Data3 data3 = new Data3();

        new Thread(()->{ for (int i = 0; i < 10; i++) data3.printA();}, "线程1").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) data3.printB();}, "线程2").start();
        new Thread(()->{ for (int i = 0; i < 10; i++) data3.printC();}, "线程3").start();
    }
}


class Data3 {

    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number = 1;  // 1A 2B 3C

    public void printA() {
        lock.lock();

        try {
            // 业务-判断-执行-通知
            while(number != 1) {
                condition1.await();
            }
            number = 2;
            System.out.println(Thread.currentThread().getName() + "=> A...");
            // 唤醒指定的人
            condition2.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void printB() {
        lock.lock();

        try {
            // 业务-判断-执行-通知
            while(number != 2) {
                condition2.await();
            }
            number = 3;
            System.out.println(Thread.currentThread().getName() + "=> B...");
            condition3.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();

        try {
            // 业务-判断-执行-通知
            while(number != 3) {
                condition3.await();
            }
            number = 1;
            System.out.println(Thread.currentThread().getName() + "=> C...");
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }


}
```



# 5. 8锁的现象

如何判断锁的是谁！对象 Class

- 总结
  - new this 具体的一个对象
  - static Class 唯一的一个模板

# 6. 集合类不安全

## 6.1. List不安全

```java
package com.xiaofan.unsafe;

import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

// java.util.ConcurrentModificationException    并发修改异常！
public class ListTest {
    public static void main(String[] args) {
        // 单线程安全
        /*List<String> list = Arrays.asList("1", "2", "3");
        list.forEach(System.out::println);*/

        // 并发下ArrayList不安全的吗？synchronized：
        /**
         * 解决方案：
         * 1. List<String> list = new Vector<>();   Vector是1.0出来的，List是1.2出来的
         * 2. List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3. List<String> list = new CopyOnWriteArrayList<>();
         */
        // CopyOnWrite 写入时复制 COW 计算机程序设计的一种优化策略
        // 多个线程调用的时候，list读取的时候，固定的，写入（覆盖）， 在写入的时候避免覆盖，造成数据问题！读写分离
        // CopyOnWriteArrayList 比 Vector牛逼的地方是底层应用了Lock锁，而Vector应用了synchronized
        List<String> list = new CopyOnWriteArrayList<>();

        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}
```

## 6.2. Set不安全

```java
package com.xiaofan.unsafe;

import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.CopyOnWriteArraySet;

// 同理：java.util.ConcurrentModificationException    并发修改异常！
public class SetTest {
    public static void main(String[] args) {

        /**
         * 解决方案：
         * 1. Set<String>set = Collections.synchronizedSet(new HashSet<>());
         * 2. Set<String>set = new CopyOnWriteArraySet<>();
         */
        Set<String>set = new CopyOnWriteArraySet<>();
        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }
}
```

- HashSet底层是什么？

```java
public HashSet() {
    map = new HashMap<>();
}

// add set 本质就是map的key是无法重复的！
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
// 不变的值
private static final Object PRESENT = new Object();
```

## 6.3. Map不安全

```java
package com.xiaofan.unsafe;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;


// java.util.ConcurrentModificationException 并发修改异常
public class MapTest {
    public static void main(String[] args) {

        /**
         * 解决方案：
         * 1. Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
         * 2. Map<String, String> map = new ConcurrentHashMap<>();
         */
        Map<String, String> map = Collections.synchronizedMap(new HashMap<>());

        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 5));
                System.out.println(map);
            }).start();
        }
    }
}
```

## 6.4. HashMap数据结构及2的整数次幂探究

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155145922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155207651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155454255.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155553534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155624151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155700313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091815592995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160021486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091816011075.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091816015935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## 6.5. HashMap加载因子及转红黑树探究

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160350830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160423480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091816045854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/202009181607310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160753852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160855957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918160939419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161017717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161038948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161248483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161304284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161427885.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091816145484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918161522338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## 6.6. ConcurrentHashMap的原理

# 7. Callable

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918164549815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 可以有返回值
- 可以抛出异常
- 方法不容，run() / call()

```java
package com.xiaofan.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
  		// 注意：同一个FutureTask下有复用情况
        FutureTask<String> futureTask1 = new FutureTask<>(new MyThread());
        FutureTask<String> futureTask2 = new FutureTask<>(new MyThread());
        new Thread(futureTask1, "A").start();
        new Thread(futureTask2, "B").start();    // 结果会被缓存，效率高

        String result1 = futureTask1.get();       // 这个方法可能会产生阻塞，把他放到最后，或者使用异步通信来处理
        System.out.println(result1);

        String result2 = futureTask1.get();
        System.out.println(result2);
    }
}


class MyThread implements Callable<String> {

    @Override
    public String call() {
        System.out.println(Thread.currentThread().getName() + " call...");
        return "1024";
    }
}
```

# 8. 常用辅助类

- [循环屏障CyclicBarrier以及和CountDownLatch的区别](https://www.cnblogs.com/twoheads/p/9555867.html)

- 从javadoc的描述可以得出：
  - CountDownLatch：一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；
  - CyclicBarrier：多个线程互相等待，直到到达同一个同步点，再继续**一起执行**。

对于CountDownLatch来说，重点是“一个线程（多个线程）等待”，而其他的N个线程在完成“某件事情”之后，可以终止，也可以等待。而对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须互相等待，然后继续一起执行。

CountDownLatch是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而CyclicBarrier更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

# ## 8.1.CountDownLatch

**减计数器**

```java
package com.xiaofan;

import java.util.concurrent.CountDownLatch;

// 计数器
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 总数是6，必须要执行任务的时候，再使用！
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + " go out...");
                countDownLatch.countDown();     // 数量减1
            }, String.valueOf(i)).start();
        }

        countDownLatch.await();     // 等待计数归零，再继续向下执行
        System.out.println("close door...");
    }
}
```



# ## 8.2. CyclicBarrier

**加法计数器**

```java
package com.xiaofan;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        /**
         * 集齐7颗龙珠召唤神龙
         */
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("召唤神龙成功！");
        });

        for (int i = 1; i <= 7; i++) {
            int temp = i;
            // Lambda 操作不到i
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + "收集" + temp + " 个龙珠");

                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                } catch (Exception e) {

                }

                System.out.println(Thread.currentThread().getName() + "continue...");
            }).start();
        }
    }
}
```

# ## 8.3. Semaphore

**信号量**

抢车位  6辆车同时抢三个车位

```java
package com.xiaofan.util;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class SemaphoreDemo {
    public static void main(String[] args) {
        // 线程数量， 停车位， 限流！
        Semaphore semaphore = new Semaphore(3);

        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                // acquire() 得到信号量
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName() + "离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();    // 释放信号量
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

**原理：**

`semaphore.acquire();`获得信号量，假如满了，等待被释放为止

`semaphore.release();` 释放信号量， 唤醒等待的线程

作用：多个共享资源互斥的使用！





# 9. 读写锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918190315346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```java
package com.xiaofan.rw;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 独占锁（写锁） 一次只能被一个线程占有
 * 共享锁（读锁） 多个线程可以同时占有
 * ReadWriteLock
 * 读-读  可以共存！
 * 读-写  不能共存！
 * 写-写  不能共存！
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
//        MyCache myCache = new MyCache();
        MyCacheLock myCacheLock = new MyCacheLock();

        for (int i = 1; i <= 5; i++) {
            int temp = i;
            new Thread(()->{
                myCacheLock.put(String.valueOf(temp), String.valueOf(temp));
            }, String.valueOf(i)).start();
        }

        for (int i = 1; i <= 5; i++) {
            int temp = i;
            new Thread(()->{
                myCacheLock.get(String.valueOf(temp));
            }, String.valueOf(i)).start();
        }
    }
}

/**
 * 自定义缓存
 */
class MyCache {

    private volatile Map<String, Object> map = new HashMap<>();

    // 存， 写
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() +" 写入 " + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() +" 写入完毕！");
    }

    // 取，读
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + " 读取 " + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + " 读取完毕！");
    }
}

/**
 * 加锁的自定义缓存
 */
class MyCacheLock {

    private volatile Map<String, Object> map = new HashMap<>();
    // 读写锁，更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    // 存， 写入的时候
    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() +" 写入 " + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() +" 写入完毕！");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    // 取，读
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " 读取 " + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + " 读取完毕！");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```



# 10. 阻塞队列

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918193612804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918194101908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918193511372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918194311141.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



什么情况下我们会使用阻塞队列： 多线程并发处理，线程池！

**四组API**（一列一列地看）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920100220103.png#pic_center)

1. 抛出异常

2. 不会抛出异常

3. 阻塞等待

4. 超时等待

```java
package com.xiaofan.bq;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.TimeUnit;

public class Test {
    public static void main(String[] args) throws InterruptedException {
        test4();
    }

    /**
     * 抛出异常
     */
    public static void test1() {
        ArrayBlockingQueue arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(arrayBlockingQueue.add(1));
        System.out.println(arrayBlockingQueue.add(2));
        System.out.println(arrayBlockingQueue.add(3));

        // java.lang.IllegalStateException: Queue full
        // System.out.println(arrayBlockingQueue.add(4));
        // 判断队首元素
        System.out.println(arrayBlockingQueue.element());

        System.out.println(arrayBlockingQueue.remove());
        System.out.println(arrayBlockingQueue.remove());
        System.out.println(arrayBlockingQueue.remove());
        // java.util.NoSuchElementException
        // System.out.println(arrayBlockingQueue.remove());

    }

    /**
     * 不抛出异常
     */
    public static void test2() {
        ArrayBlockingQueue arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(arrayBlockingQueue.offer(1));
        System.out.println(arrayBlockingQueue.offer(2));
        System.out.println(arrayBlockingQueue.offer(3));

         System.out.println(arrayBlockingQueue.offer(4));

        // 判断队首元素
        System.out.println(arrayBlockingQueue.peek());

        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        // java.util.NoSuchElementException
        // System.out.println(arrayBlockingQueue.remove());

    }

    /**
     * 等待阻塞(一直阻塞)
     */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        arrayBlockingQueue.put(1);
        arrayBlockingQueue.put(2);
        arrayBlockingQueue.put(3);
        // 等待阻塞，一直阻塞，直到放进去位置
        // arrayBlockingQueue.put(4);

        System.out.println(arrayBlockingQueue.take());
        System.out.println(arrayBlockingQueue.take());
        System.out.println(arrayBlockingQueue.take());
        // 等待阻塞，一直阻塞，直到取到元素为止
        System.out.println(arrayBlockingQueue.take());

    }

    /**
     * 等待阻塞(超时等待)
     */
    public static void test4() throws InterruptedException {
        ArrayBlockingQueue arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(arrayBlockingQueue.offer(1));
        System.out.println(arrayBlockingQueue.offer(2));
        System.out.println(arrayBlockingQueue.offer(3));
        // 超过3秒钟后则产生结果
        System.out.println(arrayBlockingQueue.offer(4, 3, TimeUnit.SECONDS));

        // 判断队首元素
        System.out.println(arrayBlockingQueue.peek());

        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll(3, TimeUnit.SECONDS));


    }
}

```

# 10.1. SynchronousQueue 同步队列

没有容量，放进去一个元素，必须等待取出来之后，才能再往里面放一个元素！

put、take

```java
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

/**
 * 同步队列
 * 和其他的BlockingQueue不一样， SynchronousQueue不存储元素
 * put进去了一个元素，必须等待take出来后才能再取！
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        SynchronousQueue<String>synchronousQueue = new SynchronousQueue();

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName() + " put 1");
                synchronousQueue.put("1");
                System.out.println(Thread.currentThread().getName() + " put 2");
                synchronousQueue.put("2");
                System.out.println(Thread.currentThread().getName() + " put 3");
                synchronousQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " " + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " " + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " " + synchronousQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```





# 11. 线程池

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920111534311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 线程池3大方法

```java
package com.xiaofan.pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

// Executors 工具类、 3大方法
public class Demo01 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();
//         ExecutorService threadPool= Executors.newCachedThreadPool();
         ExecutorService threadPool = Executors.newFixedThreadPool(5);

        try {
            for (int i = 0; i < 10; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() + " ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

- 7大参数

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

// 本质调用了ThreadPool
public ThreadPoolExecutor(int corePoolSize,		// 核心线程池大小
                          int maximumPoolSize,	// 最大线程的数量
                          long keepAliveTime,	// 超时了没有人调用就会释放
                          TimeUnit unit,		// 超时单位
                          BlockingQueue<Runnable> workQueue,	// 阻塞队列
                          ThreadFactory threadFactory,		// 线程工厂，创建线程的，一般不用动
                          RejectedExecutionHandler handler) {	// 拒绝策略
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
        null :
    AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920114623693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920114848205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 手动创建一个线程池

```java
package com.xiaofan.pool;

import java.util.concurrent.*;

/**
 * 自定义线程池
 * 没有线程可用的时候（阻塞队列也没有了，就启动拒绝策略）
 * 1.new ThreadPoolExecutor.AbortPolicy()   线程池不够用了，还有任务，就抛出异常
 * 2.new ThreadPoolExecutor.CallerRunsPolicy()  哪来的哪去
 * 3.new ThreadPoolExecutor.DiscardPolicy()   队列满了，丢掉任务，不会抛出异常
 * 4.new ThreadPoolExecutor.DiscardOldestPolicy()   队列满了，尝试和最早的竞争，也不会抛出异常
 */
public class Demo02 {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );

        try {
            for (int i = 0; i < 10; i++) {
                // 使用了线程池之后，使用线程池来创建线程
//                threadPool.execute(()->{
//                    System.out.println(Thread.currentThread().getName() + " ok");
//                });

                threadPool.execute(new MyTask(i, String.valueOf(i)));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}

class MyTask implements Runnable {

    private int taskId;
    private String taskName;

    public MyTask(int taskId, String taskName){
        this.taskId = taskId;
        this.taskName = taskName;
    }

    public int getTaskId() {
        return taskId;
    }

    public void setTaskId(int taskId) {
        this.taskId = taskId;
    }

    public String getTaskName() {
        return taskName;
    }

    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println("ID: " + this.taskId + " NAME: "+Thread.currentThread().getName() + " ok");
    }

    public String toString(){
        return Integer.toString(this.taskId);
    }

}
```



- 4种拒绝策略

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920115802445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 线程池的大小如何去设置！

```java
package com.xiaofan.pool;

import java.util.concurrent.*;

/**
 * 线程池的最大数设置
 * 1. CPU密集型，几核，就是几，可以保持CPU的效率最高！
 * 2. IO密集型，大于判断程序中十分耗IO的线程数，一半2倍即可
 */
public class Demo03 {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(2,
                Runtime.getRuntime().availableProcessors(),
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );

        try {
            for (int i = 0; i < 10; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() + " ok");
                });

            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

# 12. 四大函数式接口

新时代的攻城狮：`lambda表达式`,`连式编程`,`函数式接口`,`Stream流式计算`

函数式接口：只有一个方法的接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210203092150533.png)

- 函数式接口 Function

```java
package com.xiaofan.function;

import java.util.function.Function;

/**
 * Function 函数型接口，有一个输入参数，一个输出参数
 * 函数型接口，可以用 lambda表达式简化
 */
public class Demo01 {
    public static void main(String[] args) {
        Function<String, String> function = new Function<String, String>() {

            @Override
            public String apply(String str) {
                return str;
            }
        };

        System.out.println(function.apply("fanfan"));

        // lambda表达式
        function = str -> str;
        System.out.println(function.apply("xiaofan"));


    }
}
```

- 断定型接口 Predicate

```java
package com.xiaofan.function;

import java.util.function.Predicate;

/**
 * 断定型接口：一个输入参数，返回值只能是boolean
 */
public class Demo02 {
    public static void main(String[] args) {
        Predicate<String> predicate = new Predicate<String>() {

            @Override
            public boolean test(String s) {
                return s.isEmpty();
            }
        };

        System.out.println(predicate.test(""));

        // lambda表达式
        predicate = str -> str.isEmpty();
        System.out.println(predicate.test("fanfan"));


    }
}
```

- 供给型接口 Supplier

```java
package com.xiaofan.function;

import java.util.function.Supplier;

/**
 * 供给型接口：没有参数，只有返回值
 */
public class Demo03 {
    public static void main(String[] args) {
        Supplier<String> supplier = new Supplier<String>() {

            @Override
            public String get() {
                return "1024";
            }
        };
        System.out.println(supplier.get());

        //lambda
        supplier = ()-> "1024";
        System.out.println(supplier.get());
    }
}
```

- 消费型接口 Consumer

```java
package com.xiaofan.function;

import java.util.function.Consumer;

/**
 * 消费型接口，没有返回值，只有参数
 */
public class Demo04 {
    public static void main(String[] args) {
        Consumer<String> consumer = new Consumer<String>() {

            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };

        consumer.accept("xiaofan");
        //lambda表达式
        consumer = s -> System.out.println(s);
        consumer.accept("xiaofan");
    }
}
```





# 13. Stream流式计算

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920165221410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```java
package com.xiaofan.stream;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

/**
 * 题目要求： 一分钟内完成此题：只能用一行代码实现！
 * 现有5个用户！筛选：
 * 1. ID必须是偶数
 * 2. 年龄必须大于23岁
 * 3. 用户名转为大写字母
 * 4. 用户名字母倒着排序
 * 5. 只输出一个用户！
 */
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "b", 22);
        User u3 = new User(3, "c", 23);
        User u4 = new User(4, "d", 24);
        User u5 = new User(6, "e", 25);
        // 集合就是存储
        List<User> users = Arrays.asList(u1, u2, u3, u4, u5);
        users.stream()
                .filter(u -> u.getId() % 2 == 0)
                .filter(u -> u.getAge() > 23)
                .map(u -> u.getName().toUpperCase())
                .sorted(Comparator.reverseOrder())
                .limit(1)
                .forEach(System.out::println);
    }
}
```

# 14. 分支合并ForkJoin

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092018054970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920180717206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



```java
package com.xiaofan.forkjoin;

import java.util.concurrent.RecursiveTask;

/**
 * 求和计算的任务！
 * 3000 6000(ForkJoin) 9000(Stream并行流)
 * 如何使用forkjoin
 * 1. ForkjoinPool 通过它来执行
 * 2. 计算任务forkjoinPool.execute(ForkJoinTask task)
 * 3. 计算类要继承ForkJoinTask
 */
public class ForkJoinDemo extends RecursiveTask<Long> {

    private Long start;
    private Long end;
    private Long temp = 10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }


    @Override
    protected Long compute() {
        if ((end - start) < temp) {
            Long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {
            // forkjoin递归
            Long middle = (start + end) /2;
            // 拆分任务，把任务亚茹线程队列
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();
            // 拆分任务，把任务亚茹线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle+1, end);
            task2.fork();

            return task1.join() + task2.join();
        }
    }
}


```

- 测试

```java
package com.xiaofan.forkjoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();    // 7119
        test2();      // 5038
        test3();    // 180
    }

    public static void test1() {
        Long sum = 0L;
        long start = System.currentTimeMillis();
        for (Long i = 0L; i <= 10_1000_0000; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum= " + sum + " 时间：" + (end -start));
    }

    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinDemo forkJoinDemo = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(forkJoinDemo);// 提交任务
        Long sum = submit.get();

        long end = System.currentTimeMillis();
        System.out.println("sum= " + sum + " 时间：" + (end - start));
    }

    public static void test3() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum= " + sum + " 时间：" + (end -start));
    }

}
```



# 15. 异步回调

- 没有返回值的runAsync

```java
package com.xiaofan.feature;


import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

/**
 * 异步调用 CompletableFuture
 * 异步执行
 * 成功回调
 * 失败回调
 */
public class Demo01 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 没有返回值的runAsync 异步回调
        Future<Void> future = CompletableFuture.runAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + " runAsync => Void");
        });

        System.out.println("111");
        future.get();   // 获取阻塞执行结果
    }
}
```



- 有返回值的 supplyAsync

```java
package com.xiaofan.feature;


import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

/**
 * 异步调用 CompletableFuture
 * 异步执行
 * 成功回调
 * 失败回调
 */
public class Demo02 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 有返回值的SupplyAsync 异步回调
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + " supplyAsync => String");
//            int i = 1/0;
            return "1024";
        });

        System.out.println(future.whenComplete((t, u) -> {
            System.out.println("t=> " + t);     // t 正常的返回结果
            System.out.println("u=> " + u);     // u 错误信息
        }).exceptionally((e) -> {
            System.out.println(e.getMessage());
            return "500";
        }).get());

    }
}
```



# 16. JMM

- 什么是JMM

  **JMM： java内存模型，不存在的东西，概念，约定！**

- 关于JMM的一些同步约定：

  1. 线程解锁前，必须把共享变量**立刻**刷回主存
  2. 线程加锁前，必须读取主存中的最新值到工作内存中！
  3. 加锁和解锁必须是同一把锁

- 线程 **工作内存**、**主内存**

  - **8种操作**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921101327580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921102306822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- [Java内存模型](https://www.cnblogs.com/null-qige/p/9481900.html)





# 17. volatile

- 谈谈你对volatile的理解

  volatile是Java虚拟机提供**轻量级的同步机制**

  1. 保证可见性
  2. **不保证原子性**
  3. 禁止指令重排

- 保证可见性

```java
package com.xiaofan.tvolitale;

import java.util.concurrent.TimeUnit;

public class JMMDemo {

    // 不加volatile程序就会死循环
    // 加了volatile可以保证可见性
    private volatile static int num = 0;

    public static void main(String[] args) {
        new Thread(()->{ // 线程1 对主内存的变化不知道
            while (num == 0) {

            }
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        num = 1;
        System.out.println(num);
    }
}
```

- 不保证原子性`javap -c *.class`

原子性：不可分割

线程A在执行任务的时候，不能被打扰，也不能被分割，要么同时成功，要么同时失败

```java
package com.xiaofan.tvolitale;

public class VDemo {
    // volatile 不保证原子性
    private static int num = 0;

    public static void add() {
        num ++;
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        while(Thread.activeCount() > 2) {   // main gc
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}

```

**如果不加Lock 和synchronized，怎么保证原子性**

- 使用原子类

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921110928132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```java
package com.xiaofan.tvolitale;

import java.util.concurrent.atomic.AtomicInteger;

public class VDemo {
    // volatile 不保证原子性
    // 原子类的Integer
    private static AtomicInteger num = new AtomicInteger();

    public static void add() {
        // num ++; 不是一个原子操作
        num.getAndIncrement();  // +1 , CAS
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }

        while(Thread.activeCount() > 2) {   // main gc
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}
```

- 指令重排

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921113455110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921113628979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

**volatile指令可以避免指令编排**

内存屏障 CPU指令，作用：

1. 保证特定的操作的执行顺序
2. 可以保证某些变量的内存可见性（volatile）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921113722544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)





# 18. 深入单例模式

- 饿汉式

```java
package com.xiaofan.single;

/**
 * 饿汉式单例
 */
public class HungryMan {
    // 可能会浪费空间
    private byte[] data1 = new byte[1024*1024];
    private byte[] data2 = new byte[1024*1024];
    private byte[] data3 = new byte[1024*1024];
    private byte[] data4 = new byte[1024*1024];

    private HungryMan() {}

    private final static HungryMan HUNGRY = new HungryMan();

    public static HungryMan getInstance() {
        return HUNGRY;
    }
}
```

- 懒汉式

**双重检测**，避免不了反射破坏

```java
package com.xiaofan.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
 * 懒汉式
 */
public class LazyMan {


    private LazyMan() {
        System.out.println(Thread.currentThread().getName() + "ok");
        synchronized (LazyMan.class) {
            if (lazyMan != null) {
                throw new RuntimeException("不要视图通过反射破坏单 例！");
            }
        }

    }
    // 禁止指令重拍
    private volatile static LazyMan lazyMan;

    // 双重检测锁模式 懒汉式单例 DCL 懒汉式
    public static LazyMan getInstance() {
        if (lazyMan == null) {
            synchronized (LazyMan.class) {
                if (lazyMan == null) {
                    lazyMan = new LazyMan();    // 不是一个原子性操作
                    /**
                     * 1. 分配内存空间
                     * 2. 执行构造方法，初始化对象
                     * 3. 把这个对象指向这个空间
                     *
                     * 123
                     * 132  A
                     *      B // 此时lazyMan还没有完成构造
                     */
                }
            }
        }
        return lazyMan;
    }

    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {
//        LazyMan lazyMan1 = LazyMan.getInstance();
        // 通过反射破坏单例
        Constructor<LazyMan> constructor = LazyMan.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        LazyMan lazyMan1 = constructor.newInstance();
        LazyMan lazyMan2 = constructor.newInstance();

        System.out.println(lazyMan1);
        System.out.println(lazyMan2);
    }
}

```

- 枚举实现单例(**枚举本身就是一种类**)
  - 杜绝了反射破坏，抛出异常
  - 序列化之后，仍是同一个对象

```java
package com.xiaofan.single;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
 * 懒汉式
 */
public class LazyManApp {
    // 私有化构造方法
    private LazyManApp() {}


    //定义一个静态枚举类
    static enum SingletonEnum{
        //创建一个枚举对象，该对象天生为单例
        INSTANCE;

        private LazyManApp lazyManApp;

        //私有化枚举的构造函数
        private SingletonEnum(){
            lazyManApp = new LazyManApp();
        }
        public LazyManApp getInstance(){
            return lazyManApp;
        }
    }

    //对外暴露一个获取LazyManApp对象的静态方法
    public static LazyManApp getInstance(){
        return LazyManApp.SingletonEnum.INSTANCE.getInstance();
    }


    public static void main(String[] args) throws IOException, ClassNotFoundException, InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
        test1();
    }

    // 测试序列化
    public static void test3() throws IOException, ClassNotFoundException {
        SingletonEnum s = SingletonEnum.INSTANCE;
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("SerEnumSingleton.obj"));
        oos.writeObject(s);
        oos.flush();
        oos.close();

        FileInputStream fis = new FileInputStream("SerEnumSingleton.obj");
        ObjectInputStream ois = new ObjectInputStream(fis);
        SingletonEnum s1 = (SingletonEnum)ois.readObject();
        ois.close();
        System.out.println(s+"\n"+s1);
        System.out.println("枚举序列化前后两个是否同一个："+(s==s1));
    }

    // 测试反射
    public static void test2() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Constructor<SingletonEnum> constructor = SingletonEnum.class.getDeclaredConstructor(String.class, int.class);
        constructor.setAccessible(true);
        SingletonEnum lazyMan1 = constructor.newInstance();
        SingletonEnum lazyMan2 = constructor.newInstance();
        System.out.println(lazyMan1.getInstance());
        System.out.println(lazyMan2.getInstance());
    }

    // 测试并发
    public static void test1() {
        for (int i = 0; i < 20; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + LazyManApp.getInstance());
            }).start();
        }
    }
}
```

# 19. 深入理解CAS

- [CAS原理](https://www.jianshu.com/p/ab2c8fce878b)

```java
package com.xiaofan.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CasDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        // 期望、更新
        // public final boolean compareAndSet(int expect, int update)
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
    }
}
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921165245938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- Unsafe

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921165521424.png#pic_center)

- ABA问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921170934139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



# 20. 原子引用

> 解决ABA问题，引入原子引用，对应的思想： 乐观锁！

带版本号的原子操作！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921172309517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



```java
package com.xiaofan.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class CasSolveABADemo {

    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference(1, 1);

    public static void main(String[] args) {
        // AtomicStampedReference 注意：如果泛型是一个包装类，注意对象的引用问题
        // 正常业务操作，这里面比较的都是一个个对象

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp(); // 获取版本号
            System.out.println("a1=> " + stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("a2 => " + atomicStampedReference.compareAndSet(1, 2, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));
            System.out.println("a2 => " + atomicStampedReference.getStamp());

            System.out.println("a3 => " + atomicStampedReference.compareAndSet(2, 1, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));
            System.out.println("a3 => " + atomicStampedReference.getStamp());

        }, "a").start();


        new Thread(()->{
            int stamp = atomicStampedReference.getStamp(); // 获取版本号
            System.out.println("b1=> " + stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("b2 => " + atomicStampedReference.compareAndSet(1, 6, stamp, stamp + 1));
            System.out.println("b2 => " + atomicStampedReference.getStamp());
        }, "b").start();

    }
}
```



# 21. 可重入锁、公平锁、非公平锁、自旋锁、死锁

- 可重入锁
  - 锁套锁 对于Lock而言，加锁和解锁必须配对
- 公平、非公平锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200921175728820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 自旋锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204084354916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

- 死锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210204085638622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)

- 查看进程号：`jps -l`
- 查看堆栈信息：`jstack 进程号`

