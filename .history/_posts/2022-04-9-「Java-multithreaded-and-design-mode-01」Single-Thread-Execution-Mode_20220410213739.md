---
layout:     post
title:      "「Java multithreaded and design pattern 01」Single Threaded Execution pattern"
subtitle:   单例模式
date:       2022-04-9
author:     Leo
header-img: ""
catalog: true
tags: ["Java@Languages@Tags", "图解Java多线程设计模式@Books@Series"]
lang: zh
header-style: text
katex: true
---

有关单例模式的总结

# 概括解释

---

一次就一个共享对象，且只让一个线程访问这个共享对象。这种单线程执行(Single Threaded Execution)也可以称为**临界区(critical section)或临界域(critical region)**。

# 线程安全

---

非线程安全极难调试，每次线程的调度都是随机的，测试次数不对，时间点不对，都会造成检查的错误。并且如果显示调试信息的代码本身是非线程安全都是，输出的调试信息也有可能是错误的。(会有概率负负得正从而得到调试的信息是错的，而最后输出的是 对的)

书中建议拉人来静查。

而使用Single Threaded Execution Pattern的主要特征就在于 在多个线程访问的共享对象(一整个类也罢，或是类的一个属性也罢)，加上`synchronized`，无论是getter方法还是setter方法。

注意synchronized不会继承给子类，如果子类没有保护的会，会导致可能有的进程可以从子类访问了不安全的方法，将这个情况称为继承反常(inheritance anomaly)

## 登场角色

---

  ## 共享资源(Shared Resources)

就是所谓的共享对象，其为对象时可能有很多方法，主要可以分为safe Method和unsafe Method

其中其中unsafe Method不是线程不安全的方法，而是多个线程调用会引起冲突因此我们需要特殊考虑的方法。

而单例模式的保护方法就是将需要保护的方法声明为`synchronized`，只允许单个线程执行的程序范围可以称为临界区。

因此我们可以粗略地仍未Single Threaded Execution即为加上`synchronized`锁

## Single Threaded Execution 类图

![image.png](https://pic.rmb.bdstatic.com/bjh/7fec5e1b003b2231c1ae60a42d090081.png)

## Single Threaded Execution Pattern的线程访问

![image.png](https://pic.rmb.bdstatic.com/bjh/0dd1134bd2c2650917a527ec38facf40.png)

# 何时使用单例模式

---

显然肯定是要在多线程时再考虑。但也不是所有多线程都用：

* 如果没个线程都是完全独立的，那么就无需使用了，这种状态称为各个线程不干涉(interface)
* 当共享资源的状态不会发生变化时，也就无需采用`synchronized`

此外当我们使用**线程不安全**的容器时，也应该采用单例模式

# Single Threaded Execution 生存性和死锁

死锁也就是两个线程互相持有锁，又等待对方释放锁，导致程序无法继续运行

死锁的程序满足下列条件

1. 存在多个共享对象
2. 线程在获得其中一个锁时，还想会的另一个锁
3. 获取共享对象锁的顺序并不固定，即共享对象地位对称

举个具体例子，A和B想吃面，吃面的必要条件是勺子和叉子。如果一个人获得了勺子和叉子，则只有在它吃了一口后，才有可能放下勺子和叉子。于是如果在一个调用中，如果恰好满足类如下如下顺序，A拿着勺子、B拿着叉子，则就会导致A等着B放叉子去拿，而B等着A放下勺子去拿，从而造成死锁

```java
class Tool {
    private final String name;
    public Tool(String name) {
        this.name = name;
    }
    public String toString() {
        return "[ " + name + " ]";
    }
}



class EaterThread extends Thread {
    private String name;
    private final Tool lefthand;
    private final Tool righthand;
    public EaterThread(String name, Tool lefthand, Tool righthand) {
        this.name = name;
        this.lefthand = lefthand;
        this.righthand = righthand;
    }
    public void run() {
        while (true) {
            eat();
        }
    }
    public void eat() {
        synchronized (lefthand) {
            System.out.println(name + " takes up " + lefthand + " (left).");
            synchronized (righthand) {
                System.out.println(Thread.currentThread().getName() + " is running");
                System.out.println(name + " takes up " + righthand + " (right).");
                System.out.println(name + " is eating now, yum yum!");
                System.out.println(name + " puts down " + righthand + " (right).");
            }
            System.out.println(name + " puts down " + lefthand + " (left).");
        }
    }
}

class Main {
    public static void main(String[] args) {
        System.out.println("Testing EaterThread, hit CTRL+C to exit.");
        Tool spoon = new Tool("Spoon");
        Tool fork = new Tool("Fork");
        new EaterThread("Alice", spoon, fork).start();
        new EaterThread("Bobby", fork, spoon).start();
    }
}

```

结果如下：

![image.png](https://pic.rmb.bdstatic.com/bjh/8ee1951d42d277cb2f108f2e619fa08c.jpeg)

就出现了都死锁

破坏死锁可以按照产生死锁的情况针对性修改

1. 规定取餐具的顺序，这样就打破了SharedResources地位对称
2. 将勺子和叉子打包成一个对象，这样就是一副一副取，打破多个SharedResources

