---
layout: post
title: Java 并发编程4--Thread中start()和run()的区别
date:  2019/11/18 19:23:32 
categories: document
tag: java并发编程

---

* content
{:toc}


Thread中start()和run()的区别

# 1.start()和run()方法的区别说明 #

前面的章节中提到Thread中start()和run()方法，这节我们来看一下二者到底有何区别。

- 1.start() : 它的作用是启动一个新线程，新线程会执行相应的run()方法。start()不能被重复调用。
- 
- 2.run()   : run()就和普通的成员方法一样，可以被重复调用。单独调用run()的话，会在当前线程中执行run()，而并不会启动新线程！


下面以代码来进行说明。

```
class MyThread extends Thread{  
    public void run(){
        ...
    } 
};
MyThread mythread = new MyThread();
mythread.start();
```

mythread.start()会启动一个新线程，并在新线程中运行run()方法。
而mythread.run()则会直接在当前线程中运行run()方法，并不会启动一个新线程来运行run()。

# 2.区别示例 #

我们通过一段代码来彻底搞懂区别在哪。如下：

```
// Demo.java 的源码
class MyThread extends Thread{  
    public MyThread(String name) {
        super(name);
    }

    public void run(){
        System.out.println(Thread.currentThread().getName()+" is running");
    } 
}; 

public class Demo {  
    public static void main(String[] args) {  
        Thread mythread=new MyThread("mythread");

        System.out.println(Thread.currentThread().getName()+" call mythread.run()");
        mythread.run();

        System.out.println(Thread.currentThread().getName()+" call mythread.start()");
        mythread.start();
    }  
}

```

运行结果如下：
```
main call mythread.run()
main is running
main call mythread.start()
mythread is running
```
结果说明：

- 1.Thread.currentThread().getName()是用于获取“当前线程”的名字。当前线程是指正在cpu中调度执行的线程。
- 2.mythread.run()是在“主线程main”中调用的，该run()方法直接运行在“主线程main”上。
- 3. mythread.start()会启动“线程mythread”，“线程mythread”启动之后，会调用run()方法；此时的run()方法是运行在“线程mythread”上。

# 3.start()和run()方法的相关源码 #

Thread.java中start()方法的源码如下：

```
public synchronized void start() {
    // 如果线程不是"就绪状态"，则抛出异常！
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    // 将线程添加到ThreadGroup中
    group.add(this);

    boolean started = false;
    try {
        // 通过start0()启动线程
        start0();
        // 设置started标记
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}

```

start()实际上是通过本地方法start0()启动线程的。而start0()会新运行一个线程，新线程会调用run()方法。
```
private native void start0();
```

Thread.java中run()的代码如下：


```
public void run() {
    if (target != null) {
        target.run();
    }
}

```
说明：target是一个Runnable对象。run()就是直接调用Thread线程的Runnable成员的run()方法，并不会新建一个线程。