---
layout:     post
title:      "「OO 03」the preview of multithreaded"
subtitle:   多线程的预习
date:       2022-03-26
author:     Leo
header-img: "img/post-OO.jpg"
catalog: true
tags: ["OO@Courses@Series"]
lang: zh
header-style: text
katex: true
---



# 线程的定义

同一时刻运行多个程序的能力称为多任务(multitasking )，就打开多个程序执行多个任务的能力，比如一边打印一边玩游戏。这个在CPU的层次上称为多进程。

而多线程(multithreaded)是在较低的层次扩展了多任务的概念：**一个程序同时执行多个任务**。通常，每一个任务就称为一个线程。

多进程和多线程的区别在于，每个进程拥有自己一整套变量，和线程则**共享数据**。（和可重入程序类似的区别）尽管听起来有些风险，但共享变量会使得线程之间的通信比进程之间的通信更有效、更容易。

多线程可以完成的事情包括一个浏览器可以同时下载几幅图片；一个Web服务器需要同时处理几个并发的请求；GUI用一个独立的线程从宿主OS中收集用户界面的事件。

并行与并发：

- 并行（Parrallel）：多个cpu实例或者多台机器**同时**执行一段处理逻辑，真正的同时。
- 并发（Concurrent）：通过cpu调度算法，让用户**看上去同时**执行，实际上**从cpu操作层面不是真正的**同时。任何一个时间点实际上只有一个程序在执行。

>你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明**你不支持并发也不支持并行**。
>
>你吃饭吃到一半，电话来了，你停了下来接了电话，**接完后继续吃**饭，这说明**你支持并发**。
>
>你吃饭吃到一半，电话来了，你**一边打电话一边吃饭**，这说明你**支持并行**。
>
>并发的关键是你有处理多个任务的能力，不一定要同时。
>
>并行的关键是你有同时处理多个任务的能力。

再附上一张经典图片：

​	[![qwe1zj.png](https://s1.ax1x.com/2022/03/26/qwe1zj.png)](https://imgtu.com/i/qwe1zj)

并发和并行都可以是很多个线程，就看这些线程能不能同时被（多个）CPU执行，如果可以就说明是并行，而并发是多个线程被（一个)CPU轮流切换着执行。

- 线程安全：经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，**线程的调度顺序不影响任何结果**。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。反过来，线程不安全就意味着线程的调度顺序会影响最终结果，如不加事务的转账代码：(如果一直调用可能会导致变量没更新，从而出现转账了钱没有减少)

```java
  void transferMoney(User from, User to, float amount){
    to.setMoney(to.getBalance() + amount);
    from.setMoney(from.getBalance() - amount);
  }
```

- 同步：Java中的同步指的是**通过人为的控制和调度，保证共享资源的多线程访问成为线程安全**，来保证结果的准确。如上面的代码简单加入`@synchronized`关键字。在保证结果准确的同时，提高性能，才是优秀的程序。线程安全的优先级高于性能。

# 线程状态

---

关键词：`Thread.State`

有以下几个状态

* `New`：一个进程被创建了但还没有就绪被进行。没有`start`的话就会一直在这个状态
* `Runnable`：这个状态下，一个线程在java的虚拟机当中运行
* `Bolcked`：阻塞状态，在等待monitor lock。
* `Waiting`：无限期的等待，等待另一个线程运行
* `Timed_Waiting`：等待一个确定的时间让别的线程运行
* `Terminated`: run执行结束或stop()被调用

## 状态转移如下

----

线程状态转移如下

[![qw4Wsf.png](https://s1.ax1x.com/2022/03/27/qw4Wsf.png)](https://imgtu.com/i/qw4Wsf)

重点看一下**blocked**这个状态：
线程在Running的过程中可能会遇到阻塞(Blocked)情况

阻塞的情况分三种：

1. 等待阻塞：运行的线程执行wait()方法，该线程会释放占用的所有资源，JVM会把该线程放入“等待池中”。进入这个状态后是不能自动唤醒的，必须依靠其他线程调用notify()或者notifyAll()方法才能被唤醒。
2. 同步阻塞：运行的线程在获取对象的（synchronized）同步锁时，若该同步锁被其他线程占用，则JVM会吧该线程放入“锁池”中。
3. 其他阻塞：通过调用线程的sleep()或者join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新回到就绪状态

此外，在runnable状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread类中的yield方法可以让一个running状态的线程转入runnable。

Running的run结束、异常、stop()可以结束这个进程



​                                              

# 基本线程的创建

---

基本线程类为Thread类，Runnable接口，Callable接口

Thread 类实现了Runnable接口，启动一个线程的方法：

```java
MyThread my = new MyThread();
my.start();
```

## Thread类相关方法

---

```java
// 主要的某些常用的并不通用的方法
//当前线程可转让cpu控制权，让别的就绪状态线程运行（切换）
public static Thread.yield() 
//暂停一段时间
public static Thread.sleep()  
//在一个线程中调用other.join(),将等待other执行完后才继续本线程。　　　　
public join()
//后两个函数皆可以被打断
public interrupt()
```

**关于中断**：它并不像stop方法那样会中断一个正在运行的线程。线程会不时地检测中断标识位，以判断线程是否应该被中断（中断标识值是否为true）。中断只会影响到wait状态、sleep状态和join状态。被打断的线程会抛出`InterruptedException`。
`Thread.interrupted()`检查当前线程是否发生中断，返回`boolean` 

中断是一个状态, `interrupt()`方法只是将这个状态置为true而已。所以说正常运行的程序不去检测状态，就不会终止，而wait等阻塞方法会去检查并抛出异常。如果在正常运行的程序中添加`while(!Thread.interrupted()) `，则同样可以在中断后离开代码体

该类继承 Thread 类，然后创建一个该类的实例。

使用继承Thread类的方法实现多线程，继承类必须重写 run() 方法，该方法是新线程的入口点。它也必须调用 start() 方法才能执行。

该方法尽管被列为一种多线程实现方式，但是本质上也是实现了 Runnable 接口的一个实例。

```java
class ThreadDemo extends Thread {
   private Thread t;
   private String threadName;
   
   ThreadDemo( String name) {
      threadName = name;
      System.out.println("Creating " +  threadName );
   }
   
   public void run() {
      System.out.println("Running " +  threadName );
      try {
         for(int i = 4; i > 0; i--) {
            System.out.println("Thread: " + threadName + ", " + i);
            // 让线程睡眠一会
            Thread.sleep(50);
         }
      }catch (InterruptedException e) {
         System.out.println("Thread " +  threadName + " interrupted.");
      }
      System.out.println("Thread " +  threadName + " exiting.");
   }
   
   public void start () {
      System.out.println("Starting " +  threadName );
      if (t == null) {
         t = new Thread (this, threadName);
         t.start ();
      }
   }
}
 
public class TestThread {
 
   public static void main(String[] args) {
      ThreadDemo T1 = new ThreadDemo( "Thread-1");
      T1.start();
      
      ThreadDemo T2 = new ThreadDemo( "Thread-2");
      T2.start();
   }   
}
```

编译以上程序运行结果如下：

```sh
Creating Thread-1
Starting Thread-1
Creating Thread-2
Starting Thread-2
Running Thread-1
Thread: Thread-1, 4
Running Thread-2
Thread: Thread-2, 4
Thread: Thread-1, 3
Thread: Thread-2, 3
Thread: Thread-2, 2
Thread: Thread-1, 2
Thread: Thread-1, 1
Thread: Thread-2, 1
Thread Thread-2 exiting.
Thread Thread-1 exiting.
```

再运行一次

```sh
Creating Thread-1
Starting Thread-1
Creating Thread-2
Starting Thread-2
Running Thread-1
Running Thread-2
Thread: Thread-2, 4
Thread: Thread-1, 4
Thread: Thread-1, 3
Thread: Thread-2, 3
Thread: Thread-1, 2
Thread: Thread-2, 2
Thread: Thread-2, 1
Thread: Thread-1, 1
Thread Thread-2 exiting.
Thread Thread-1 exiting.
```

可以看到，两次并不一样，个线程对象是交错运行的，哪个线程对象抢到了 CPU 资源，哪个线程就可以运行，所以程序每次的运行结果肯定是不一样的，在线程启动虽然调用的是 start() 方法，但实际上调用的却是 run() 方法定义的主体。

## Thread 方法

下表列出了Thread类的一些重要方法：

| **序号** |                         **方法描述**                         |
| :------: | :----------------------------------------------------------: |
|    1     | `public void start()` 使该线程开始执行, 进入预备状态，Java 调用该线程的 run 方法。 |
|    2     | `public void run()` 如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，该方法不执行任何操作并返回。 |
|    3     | `public final void setName(String name)` 改变线程名称，使之与参数 name 相同。 |
|    4     | `public final void setPriority(int priority)`  更改线程的优先级。 |
|    5     | `public final void setDaemon(boolean on)` 将该线程标记为守护线程或用户线程，后台线程。 |
|    6     | `public final void join(long millisec)`等待该线程终止的时间最长为 millis 毫秒。如果不佳millisec则强制运行线程至结束，强制运行时其他线程无法运行 |
|    7     |             `public void interrupt()` 中断线程。             |
|    8     | `public final boolean isAlive()` 测试线程是否处于活动状态。  |
|    9     |          `public String getName()`：获取线程的名字           |

上述方法是被 Thread 对象调用的，下面表格的方法是 Thread 类的静态方法。

| **序号** |                         **方法描述**                         |
| :------: | :----------------------------------------------------------: |
|    1     | `public static void yield()` 暂停当前正在执行的线程对象，并执行其他线程。 |
|    2     | `public static void sleep(long millisec)` 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。 |
|    3     | `public static boolean holdsLock(Object x)` 当且仅当当前线程在指定的对象上保持监视器锁时，才返回 true。 |
|    4     | `public static Thread currentThread()` 返回对当前正在执行的线程对象的引用。 |
|    5     | `public static void dumpStack()` 将当前线程的堆栈跟踪打印至标准错误流。 |

## Runnable 接口

---

应该是实现多线程最简单的方法

一个类需要实现一个run的方法

```java
public void run()
```

可以重写该方法，重要的是理解的 **run() 可以调用其他方法，使用其他类，并声明变量，就像主线程一样**。

在创建一个实现 Runnable 接口的类之后，可以在类中实例化一个线程对象。

Thread 定义了几个构造方法，下面的这个是我们经常使用的：

`Thread(Runnable threadOb[, String threadName])`;

这里，threadOb 是一个实现 Runnable 接口的类的实例，并且 threadName 指定新线程的名字。

这是于继承thread略微有区别的地方。

新线程创建之后，你调用它的 start() 方法它才会运行。

`void start();`

然后就是会执行run里面的东西

下面是一个创建线程并开始让它执行的实例：

```java
class RunnableDemo implements Runnable {
   private Thread t;
   private String threadName;
   
   RunnableDemo( String name) {
      threadName = name;
      System.out.println("Creating " +  threadName );
   }
   
   public void run() {
      System.out.println("Running " +  threadName );
      try {
         for(int i = 4; i > 0; i--) {
            System.out.println("Thread: " + threadName + ", " + i);
            // 让线程睡眠一会
            Thread.sleep(50);
         }
      }catch (InterruptedException e) {
         System.out.println("Thread " +  threadName + " interrupted.");
      }
      System.out.println("Thread " +  threadName + " exiting.");
   }
   
   public void start () {
      System.out.println("Starting " +  threadName );
      if (t == null) {
         t = new Thread (this, threadName);
         t.start ();
      }
   }
}
 
public class TestThread {
    
    public static void main(String[] args) {
        RunnableDemo R1 = new RunnableDemo( "Thread-1");
        Thread t1 = new Thread(R1);
        t1.start();
        RunnableDemo R2 = new RunnableDemo( "Thread-2");
        Thread t2 = new Thread(R2);
        t2.start();
    }
}
```

编译以上程序运行结果如下：

```sh
Creating Thread-1
Starting Thread-1
Creating Thread-2
Starting Thread-2
Running Thread-1
Thread: Thread-1, 4
Running Thread-2
Thread: Thread-2, 4
Thread: Thread-1, 3
Thread: Thread-2, 3
Thread: Thread-1, 2
Thread: Thread-2, 2
Thread: Thread-1, 1
Thread: Thread-2, 1
Thread Thread-1 exiting.
Thread Thread-2 exiting.
```

## `Callable` 和 `future`

callable不仅实现了多线程，还可以具有返回值(Runnable or Thread如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果)

- 创建 `Callable `接口的实现类，并实现 `call() `方法，该 `call() `方法将作为线程执行体，并且有返回值。
- 创建 `Callable `实现类的实例，使用 `FutureTask` 类来包装 Callable 对象，该 FutureTask 对象封装了该` Callable `对象的 `call()` 方法的返回值。
- 使用 `FutureTask` 对象作为 Thread 对象的 target 创建并启动新线程。
- 调用 `FutureTask` 对象的 `get() `方法来获得子线程执行结束后的返回值。

除此之外，future还有别的方法

Future是一个接口，它可以对Callable任务的执行结果进行操作。可以说Future提供了三种功能：判断任务是否完成；能够中断任务；能够获取任务执行结果。

```java
boolean cancel(boolean mayInterruptIfRunning);
boolean isCancelled();
boolean isDone();
V get() throws InterruptedException, ExecutionException;
V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
```
* `cancel()`方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。

  参数`mayInterruptIfRunning`表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。

  如果任务已经完成，则无论`mayInterruptIfRunning`为true还是false，此方法**必然返回false**，即如果取消已经完成的任务会返回false；

  如果任务正在执行，若`mayInterruptIfRunning`设置为true，则返回true，若`mayInterruptIfRunning`设置为false，则返回false；

  如果任务还没有执行，则无论`mayInterruptIfRunning`为true还是false，肯定返回true。

* `isCancelled()`方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

* `isDone()`方法表示任务是否已经完成，若任务完成，则返回true；

* `get()`方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

* `get(long timeout, TimeUnit unit)`用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

**示例：**

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableThreadTest implements Callable<Integer> {
    public static void main(String[] args)
    {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);
        for(int i = 0;i < 100;i++)
        {
            System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
            if(i==20)
            {
                new Thread(ft,"New Thread with return value").start();
            }
        }
        try
        {
            System.out.println("子线程的返回值："+ft.get());
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        } catch (ExecutionException e)
        {
            e.printStackTrace();
        }
        
    }
    
    @Override
    public Integer call() throws Exception
    {
        int i = 0;
        for(;i<100;i++)
        {
            System.out.println(Thread.currentThread().getName()+" "+i);
        }
        return i;
    }
}
```



```sh
main 的循环变量i的值0
main 的循环变量i的值1
main 的循环变量i的值2
main 的循环变量i的值3
main 的循环变量i的值4
main 的循环变量i的值5
main 的循环变量i的值6
main 的循环变量i的值7
main 的循环变量i的值8
main 的循环变量i的值9
main 的循环变量i的值10
main 的循环变量i的值11
main 的循环变量i的值12
main 的循环变量i的值13
main 的循环变量i的值14
main 的循环变量i的值15
main 的循环变量i的值16
main 的循环变量i的值17
main 的循环变量i的值18
main 的循环变量i的值19
main 的循环变量i的值20
main 的循环变量i的值21
main 的循环变量i的值22
main 的循环变量i的值23
main 的循环变量i的值24
main 的循环变量i的值25
main 的循环变量i的值26
main 的循环变量i的值27
main 的循环变量i的值28
New Thread with return value 0
main 的循环变量i的值29
New Thread with return value 1
main 的循环变量i的值30
New Thread with return value 2
main 的循环变量i的值31
New Thread with return value 3
main 的循环变量i的值32
New Thread with return value 4
main 的循环变量i的值33
New Thread with return value 5
main 的循环变量i的值34
New Thread with return value 6
main 的循环变量i的值35
New Thread with return value 7
main 的循环变量i的值36
New Thread with return value 8
main 的循环变量i的值37
main 的循环变量i的值38
main 的循环变量i的值39
New Thread with return value 9
main 的循环变量i的值40
New Thread with return value 10
main 的循环变量i的值41
New Thread with return value 11
main 的循环变量i的值42
New Thread with return value 12
main 的循环变量i的值43
main 的循环变量i的值44
main 的循环变量i的值45
main 的循环变量i的值46
main 的循环变量i的值47
New Thread with return value 13
main 的循环变量i的值48
New Thread with return value 14
main 的循环变量i的值49
New Thread with return value 15
main 的循环变量i的值50
New Thread with return value 16
main 的循环变量i的值51
New Thread with return value 17
main 的循环变量i的值52
New Thread with return value 18
main 的循环变量i的值53
New Thread with return value 19
main 的循环变量i的值54
New Thread with return value 20
main 的循环变量i的值55
New Thread with return value 21
main 的循环变量i的值56
New Thread with return value 22
main 的循环变量i的值57
New Thread with return value 23
main 的循环变量i的值58
New Thread with return value 24
New Thread with return value 25
main 的循环变量i的值59
New Thread with return value 26
main 的循环变量i的值60
New Thread with return value 27
main 的循环变量i的值61
New Thread with return value 28
main 的循环变量i的值62
New Thread with return value 29
main 的循环变量i的值63
New Thread with return value 30
main 的循环变量i的值64
New Thread with return value 31
main 的循环变量i的值65
New Thread with return value 32
main 的循环变量i的值66
New Thread with return value 33
main 的循环变量i的值67
New Thread with return value 34
main 的循环变量i的值68
New Thread with return value 35
main 的循环变量i的值69
New Thread with return value 36
main 的循环变量i的值70
New Thread with return value 37
main 的循环变量i的值71
New Thread with return value 38
main 的循环变量i的值72
New Thread with return value 39
main 的循环变量i的值73
New Thread with return value 40
main 的循环变量i的值74
New Thread with return value 41
main 的循环变量i的值75
New Thread with return value 42
New Thread with return value 43
main 的循环变量i的值76
New Thread with return value 44
main 的循环变量i的值77
New Thread with return value 45
main 的循环变量i的值78
New Thread with return value 46
main 的循环变量i的值79
New Thread with return value 47
New Thread with return value 48
main 的循环变量i的值80
New Thread with return value 49
main 的循环变量i的值81
New Thread with return value 50
main 的循环变量i的值82
New Thread with return value 51
main 的循环变量i的值83
New Thread with return value 52
main 的循环变量i的值84
New Thread with return value 53
main 的循环变量i的值85
New Thread with return value 54
main 的循环变量i的值86
New Thread with return value 55
main 的循环变量i的值87
New Thread with return value 56
main 的循环变量i的值88
New Thread with return value 57
main 的循环变量i的值89
New Thread with return value 58
main 的循环变量i的值90
New Thread with return value 59
main 的循环变量i的值91
New Thread with return value 60
main 的循环变量i的值92
New Thread with return value 61
main 的循环变量i的值93
New Thread with return value 62
main 的循环变量i的值94
New Thread with return value 63
main 的循环变量i的值95
main 的循环变量i的值96
New Thread with return value 64
main 的循环变量i的值97
New Thread with return value 65
New Thread with return value 66
New Thread with return value 67
New Thread with return value 68
New Thread with return value 69
main 的循环变量i的值98
main 的循环变量i的值99
New Thread with return value 70
New Thread with return value 71
New Thread with return value 72
New Thread with return value 73
New Thread with return value 74
New Thread with return value 75
New Thread with return value 76
New Thread with return value 77
New Thread with return value 78
New Thread with return value 79
New Thread with return value 80
New Thread with return value 81
New Thread with return value 82
New Thread with return value 83
New Thread with return value 84
New Thread with return value 85
New Thread with return value 86
New Thread with return value 87
New Thread with return value 88
New Thread with return value 89
New Thread with return value 90
New Thread with return value 91
New Thread with return value 92
New Thread with return value 93
New Thread with return value 94
New Thread with return value 95
New Thread with return value 96
New Thread with return value 97
New Thread with return value 98
New Thread with return value 99
子线程的返回值：100
```

交错着出现就是说明两个进程在互相争夺着CPU

## 方法比较

- 采用实现 Runnable、Callable 接口的方式创建多线程时，线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。
- 使用继承 Thread 类的方式创建多线程时，编写简单，如果需要访问当前线程，则无需使用 `Thread.currentThread() `方法，直接使用 this 即可获得当前线程。

# 线程的一些简单的操作方法

## 强制运行

利用`join()`可以

## 中断程序

```java
public class MyThread implements Runnable{
    public void run(){  // 覆写run()方法
        System.out.println("1、进入run()方法") ;
        try{
            Thread.sleep(10000) ;   // 线程休眠10秒
            System.out.println("2、已经完成了休眠") ;
        }catch(InterruptedException e){
            System.out.println("In the class, sleep") ;
            return ; // 返回调用处
        }
        System.out.println("4、run()方法正常结束") ;
    }
}

public class ThreadInterruptDemo {
    public static void main(String[] args){
        MyThread mt = new MyThread() ;  // 实例化Runnable子类对象
        Thread t = new Thread(mt,"线程");     // 实例化Thread对象
        t.start() ; // 启动线程
        try{
            Thread.sleep(2000) ;// 线程休眠2秒
            //System.out.println("In the main");
        }catch(InterruptedException e){
            System.out.println("It has been sleeping") ;
        }
        t.interrupt() ; // 中断线程执行
    }
}

```

输出：

```sh
1、进入run()方法
In the class, sleep
```

## 后台运行

在 Java 程序中，只要前台有一个线程在运行，则整个 Java 进程都不会消失，所以此时可以设置一个后台线程，这样即使 Java 线程结束了，此后台线程依然会继续执行，要想实现这样的操作，直接使用 `setDaemon()` 方法即可。用法`mythread.setDaemon(true)`

## 优先级

从之前的例子也可以看出，多个线程处在runnable状态时，谁抢到CPU资源完全是随机事件。如果我们硬要让每个进程按一定的顺序执行，可以设置其优先级，利用`THread_MIN_PRIORITY`, `Thread.MAX_PRIORITY`, `Thread.NORM_PRIORITY`设置

```java
class MyThread implements Runnable{ // 实现Runnable接口
    public void run(){  // 覆写run()方法
        for(int i=0;i<5;i++){
            try{
                Thread.sleep(500) ; // 线程休眠
            }catch(InterruptedException e){
            }
            System.out.println(Thread.currentThread().getName()
                    + "运行，i = " + i) ;  // 取得当前线程的名字
        }
    }
};
public class ThreadPriorityDemo{
    public static void main(String args[]){
        Thread t1 = new Thread(new MyThread(),"线程A") ;  // 实例化线程对象
        Thread t2 = new Thread(new MyThread(),"线程B") ;  // 实例化线程对象
        Thread t3 = new Thread(new MyThread(),"线程C") ;  // 实例化线程对象
        t1.setPriority(Thread.MIN_PRIORITY) ;   // 优先级最低
        t2.setPriority(Thread.MAX_PRIORITY) ;   // 优先级最高
        t3.setPriority(Thread.NORM_PRIORITY) ;  // 优先级最中等
        t1.start() ;    // 启动线程
        t2.start() ;    // 启动线程
        t3.start() ;    // 启动线程
    }
};

/* 
Output：
线程B运行，i = 0
线程C运行，i = 0
线程A运行，i = 0
线程B运行，i = 1
线程C运行，i = 1
线程A运行，i = 1
线程B运行，i = 2
线程C运行，i = 2
线程A运行，i = 2
线程B运行，i = 3
线程C运行，i = 3
线程A运行，i = 3
线程B运行，i = 4
线程C运行，i = 4
线程A运行，i = 4
*/

```



# 多线程机制

---

## Monitor

[![qw5ky6.png](https://s1.ax1x.com/2022/03/27/qw5ky6.png)](https://imgtu.com/i/qw5ky6)

Monitor是应用于同步问题的人工线程调度工具。讲其本质，Java中的每个对象都有一个监视器，来监测并发代码的重入。在非多线程编码时该监视器不发挥作用，反之如果在synchronized 范围内，监视器发挥作用。

`wait/notify`必须存在于synchronized块中。并且，这三个关键字针对的是同一个监视器（某对象的监视器）。这意味着wait之后，其他线程可以进入同步块执行。当执行结束时，调用`notify()/notifyAll()`可以

当某代码并不持有监视器的使用权时（如图中5的状态，即脱离同步块）,如果能要用`wait\notify`，会抛出`java.lang.IllegalMonitorStateException`。这个异常也会在synchronized块中去调用另一个对象的`wait/notify`时抛出，因为不同对象的监视器不同

## 同步与死锁

### `sychronized()`单独使用

可以利用sychronized将类的属性进行同步

代码块：如下，在多线程环境下，synchronized块中的方法获取了lock实例的monitor，如果实例相同，那么只有一个线程能执行该块内容

```java
public class Thread1 implements Runnable {
   Object lock; // the object need to be sychrnoized
   public void run() {  
       synchronized(lock){
         /*do something
          * the code need to be sychrnoized
          */ 
       }
   }
}
```

直接用于方法： 相当于上面代码中用lock来锁定的效果，实际获取的是Thread1类的monitor。

```java
public class Thread1 implements Runnable {
   public synchronized void run() {  
        //do something
   }
}
```

典型场景生产者消费者问题

```java
/**
   * 生产者生产出来的产品交给店员
   */
  public synchronized void produce()
  {
      if(this.product >= MAX_PRODUCT)
      {
          try
          {
              wait();  
              System.out.println("产品已满,请稍候再生产");
          }
          catch(InterruptedException e)
          {
              e.printStackTrace();
          }
          return;
      }

      this.product++;
      System.out.println("生产者生产第" + this.product + "个产品.");
      notifyAll();   //通知等待区的消费者可以取出产品了
  }

  /**
   * 消费者从店员取产品
   */
  public synchronized void consume()
  {
      if(this.product <= MIN_PRODUCT)
      {
          try 
          {
              wait(); 
              System.out.println("缺货,稍候再取");
          } 
          catch (InterruptedException e) 
          {
              e.printStackTrace();
          }
          return;
      }

      System.out.println("消费者取走了第" + this.product + "个产品.");
      this.product--;
      notifyAll();   //通知等待去的生产者可以生产产品了
  }
```

应该在语句块执行结束浅通过`notify()/notify()`通知JVM

更进一步，如果修饰的是`static`方法，则**锁定该类所有实例**，即对象就是class(one thread per class)。

`synchronized`在获锁的过程中是不能被中断的。

## 死锁

同步可以保证资源共享操作的正确性，但是过多同步也会产生问题。例如，现在张三想要李四的画，李四想要张三的书，张三对李四说“把你的画给我，我就给你书”，李四也对张三说“把你的书给我，我就给你画”两个人互相等对方先行动，就这么干等没有结果，这实际上就是死锁的概念。

所谓死锁，就是两个线程都在等待对方先完成，造成程序的停滞，一般程序的死锁都是在程序运行时出现的。

## volatile

---

多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存load到本地栈，完成操作后再save回去。

volatile关键词的**作用**：每次针对该变量的操作都激发一次load & save。针对多线程使用的变量如果不是volatile或者final修饰的，很有可能产生不可预知的结果，比如另一个线程修改了这个值，但是之后在某线程看到的是修改之前的值（可重入程序出现bug的原因之一）。其实道理上讲，同一实例的同一属性本身只有一个副本。但是多线程是会缓存值的（因此有可能缓存到之前的值，导致时光回溯了），本质上，volatile就是不去缓存，直接取值。在线程安全的情况下加volatile会牺牲性能。