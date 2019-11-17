---
layout: post
title: Java 并发编程2--重要概念及如何创建并运行java线程 
date:  2019/11/17 16:35:36 
categories: document
tag: java并发编程

---

* content
{:toc}








重要概念及如何创建并运行java线程

# 1.几个重要概念 #

## 1.1同步和异步 ##

同步和异步通常用来形容一次方法调用。同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者可以继续后续的操作。


关于异步目前比较经典以及常用的实现方式就是消息队列：在不使用消息队列服务器的时候，用户的请求数据直接写入数据库，在高并发的情况下数据库压力剧增，使得响应速度变慢。但是在使用消息队列之后，用户的请求数据发送给消息队列之后立即 返回，再由消息队列的消费者进程从消息队列中获取数据，异步写入数据库。由于消息队列服务器处理速度快于数据库（消息队列也比数据库有更好的伸缩性），因此响应速度得到大幅改善。


##1.2并发(Concurrency)和并行(Parallelism)##

并发和并行是两个非常容易被混淆的概念。它们都可以表示两个或者多个任务一起执行，但是偏重点有些不同。并发偏重于多个任务交替执行，而多个任务之间有可能还是串行的。而并行是真正意义上的“同时执行”。

多线程在单核CPU的话是顺序执行，也就是交替运行（并发）。多核CPU的话，因为每个CPU有自己的运算器，所以在多个CPU中可以同时运行（并行）。

##1.3 高并发##

高并发（High Concurrency）是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计保证系统能够同时并行处理很多请求。

高并发相关常用的一些指标有响应时间（Response Time），吞吐量（Throughput），每秒查询率QPS（Query Per Second），并发用户数等。
##1.4 临界区##

临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。在并行程序中，临界区资源是保护的对象。

##1.5 阻塞和非阻塞##

非阻塞指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回，而阻塞与之相反.

# 2.创建java线程的三种方式 #

##2.1继承Thread类##
创建Thread子类的一个实例并重写run方法，run方法会在调用start()方法之后被执行。例子如下：

```
public class MyThread extends Thread {
	@Override
	public void run() {
		System.out.println("MyThread");
	}
}
```

可以用如下方式创建并运行上述Thread子类
```
public class Run {

	public static void main(String[] args) {
		MyThread mythread = new MyThread();
		mythread.start();
		System.out.println("运行结束");
	}

}
//结果为：
//运行结束
//MyThread
```

从上面的运行结果可以看出：一旦线程启动后start方法就会立即返回，而不会等待到run方法执行完毕才返回。就好像run方法是在另外一个cpu上执行一样。当run方法执行后，将会打印出字符串MyThread running。 

也可以如下创建一个Thread的匿名子类：
```
    Thread thread = new Thread(){  

	   @Override
       public void run(){  
         System.out.println("Thread Running");  
       }  
    };  
    thread.start();  
```



## 2.2实现Runnable接口 ##
第二种编写线程执行代码的方式是新建一个实现了java.lang.Runnable接口的类的实例，实例中的方法可以被线程调用。推荐实现Runnable接口方式开发多线程，因为Java单继承但是可以实现多个接口。下面给出例子： 

```
public class MyRunnable implements Runnable {
	@Override
	public void run() {
		System.out.println("MyRunnable");
	}
}

```
为了使线程能够执行run()方法，需要在Thread类的构造函数中传入 MyRunnable的实例对象。
```
public class Run {

	public static void main(String[] args) {
		Runnable runnable=new MyRunnable();
		Thread thread=new Thread(runnable);
		thread.start();
		System.out.println("运行结束！");
	}

}

```
当线程运行时，它将会调用实现了Runnable接口的run方法。上例中将会打印出“MyRunnable running”。


同样，也可以创建一个实现了Runnable接口的匿名类，如下所示：

```
    Runnable myRunnable = new Runnable(){  
       public void run(){  
         System.out.println("Runnable running");  
       }  
    };  
    Thread thread = new Thread(myRunnable);  
    thread.start();  

```

## 2.3使用线程池 ##

使用线程池的方式也是最推荐的一种方式，另外，《阿里巴巴Java开发手册》在第一章第六节并发处理这一部分也强调到“线程资源必须通过线程池提供，不允许在应用中自行显示创建线程”。这里就不给大家演示代码了，线程池这一节会详细介绍到这部分内容。


# 3.创建子类还是实现Runnable接口？ #

对于这两种方式哪种好并没有一个确定的答案，它们都能满足要求。就我个人意见，我更倾向于实现Runnable接口这种方法。因为线程池可以有效的管理实现了Runnable接口的线程，如果线程池满了，新的线程就会排队等候执行，直到线程池空闲出来为止。而如果线程是通过实现Thread子类实现的，这将会复杂一些。

有时我们要同时融合实现Runnable接口和Thread子类两种方式。例如，实现了Thread子类的实例可以执行多个实现了Runnable接口的线程。一个典型的应用就是线程池。


**常见错误：调用run()方法而非start()方法**

创建并运行一个线程所犯的常见错误是调用线程的run()方法而非start()方法，如下所示：

Java代码  收藏代码
```
    Thread newThread = new Thread(MyRunnable());  
    newThread.run();  //should be start();  
```
起初你并不会感觉到有什么不妥，因为run()方法的确如你所愿的被调用了。但是，事实上,run()方法并非是由刚创建的新线程所执行的，而是被创建新线程的当前线程所执行了。也就是被执行上面两行代码的线程所执行的。想要让创建的新线程执行run()方法，必须调用新线程的start方法。

# 4.线程的一些常用方法 # 

**currentThread()**

返回对当前正在执行的线程对象的引用。

**getId()**

返回此线程的标识符

**getName()**

返回此线程的名称

**getPriority()**

返回此线程的优先级

**isAlive()**

测试这个线程是否还处于活动状态。
**什么是活动状态呢？**
活动状态就是线程已经启动且尚未终止。线程处于正在运行或准备运行的状态。

**sleep(long millis)**

使当前正在执行的线程以指定的毫秒数“休眠”（暂时停止执行），具体取决于系统定时器和调度程序的精度和准确性。

**interrupt()**

中断这个线程。

**interrupted() 和isInterrupted()**

interrupted()：测试当前线程是否已经是中断状态，执行后具有将状态标志清除为false的功能

isInterrupted()： 测试线程Thread对相关是否已经是中断状态，但部清楚状态标志

**setName(String name)**

将此线程的名称更改为等于参数 name 。

**isDaemon()**

测试这个线程是否是守护线程。

**setDaemon(boolean on)**

将此线程标记为 daemon线程或用户线程。

**join()**

在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是 主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行

**yield()**

yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU时间。注意：放弃的时间不确定，可能一会就会重新获得CPU时间片。

**setPriority(int newPriority)**

更改此线程的优先级。

# 6.使用Sleep方法暂停一个线程 #

使用Thread.sleep()方法可以暂停当前线程一段时间。这是一种使处理器时间可以被其他线程或者运用程序使用的有效方式。sleep()方法还可以用于调整线程执行节奏和等待其他有执行时间需求的线程。

在Thread中有两个不同的sleep()方法，一个使用毫秒表示休眠的时间，而另一个是用纳秒。由于操作系统的限制休眠时间并不能保证十分精确。休眠周期可以被interrups所终止，我们将在后面看到这样的例子。不管在任何情况下，我们都不应该假定调用了sleep()方法就可以将一个线程暂停一个十分精确的时间周期。

SleepMessages程序为我们展示了使用sleep()方法每四秒打印一个信息的例子 

```
public class SleepMessages {  
    public static void main(String args[])  
        throws InterruptedException {  
        String importantInfo[] = {  
            "Mares eat oats",  
            "Does eat oats",  
            "Little lambs eat ivy",  
            "A kid will eat ivy too"  
        };  
        for (int i = 0; i < importantInfo.length;i++) {  
            //Pause for 4 seconds  
            Thread.sleep(4000);  
            //Print a message  
            System.out.println(importantInfo[i]);  
        }  
    }  
} 
```
main()方法声明了它有可能抛出InterruptedException。当其他线程中断当前线程时，sleep()方法就会抛出该异常。由于这个应用程序并没有定义其他的线程，所以并不用关心如何处理该异常。 

# 7.中断（Interrupts）一个线程 #

中断是给线程的一个指示，告诉它应该停止正在做的事并去做其他事情。一个线程究竟要怎么响应中断请求取决于程序员，不过让其终止是很普遍的做法。这是本文重点强调的用法。 

一个线程通过调用对被中断线程的Thread对象的interrupt()方法，发送中断信号。为了让中断机制正常工作，被中断的线程必须支持它自己的中断(即要自己处理中断) 

**中断支持**
线程如何支持自身的中断？这取决于它当前正在做什么。如果线程正在频繁调用会抛InterruptedException异常的方法，在捕获异常之后，它只是从run()方法中返回。例如，假设在上节的SleepMessages的例子中，关键的消息循环在线程的Runnable对象的run方法中，代码可能会被修改成下面这样以支持中断： 
```
for (int i = 0; i < importantInfo.length; i++) {  
    // Pause for 4 seconds  
    try {  
       Thread.sleep(4000);  
    } catch (InterruptedException e) {  
       // We've been interrupted: no more messages.  
      return;  
 }  
 // Print a message  
 System.out.println(importantInfo[i]);  
} 
```
许多会抛InterruptedException异常的方法(如sleep()），被设计成接收到中断后取消它们当前的操作，并在立即返回。

如果一个线程长时间运行而不调用会抛InterruptedException异常的方法会怎样？ 那它必须周期性地调用Thread.interrupted()方法，该方法在接收到中断请求后返回true。例如： 
```
    for (int i = 0; i < inputs.length; i++) {  
        heavyCrunch(inputs[i]);  
        if (Thread.interrupted()) {  
            // We've been interrupted: no more crunching.  
            return;  
        }  
    } 

``` 

在这个简单的例子中，代码只是检测中断，并在收到中断后退出线程。在更复杂的应用中，抛出一个InterruptedException异常可能更有意义。

```
    for (int i = 0; i < inputs.length; i++) {  
        heavyCrunch(inputs[i]);  
        if (Thread.interrupted()) {  
            // We've been interrupted: no more crunching.  
            throw new InterruptedException();
        }  
    } 
```

这使得中断处理代码能集中在catch语句中。 

下面的例子中，有线程池模拟了一个终端的场景。1个是基于sleep的终端，另外一个是循环检测终端的状态，抛出终端的异常。线程池
executor.shutdownNow();方法，会立刻终端所有线程。

```
    package com.chinaso.phl;  
      
    import java.util.concurrent.ExecutorService;  
    import java.util.concurrent.Executors;  
      
    public class InterruptedTest {  
        private static ExecutorService executor = Executors.newFixedThreadPool(10);  
      
        public static void main(String[] args) throws Exception {  
      
            executor.execute(new Runnable() {  
      
                @Override  
                public void run() {  
                    try {  
                        for (int i = 0; i < Integer.MAX_VALUE; i++) {  
                            if (Thread.currentThread().isInterrupted()) {  
                                throw new InterruptedException("循环被中断了");  
                            }  
                        }  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
      
            executor.execute(new Runnable() {  
                @Override  
                public void run() {  
                    try {  
                        Thread.sleep(Integer.MAX_VALUE);  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
            System.out.println("begin");  
            Thread.sleep(1000);  
            System.out.println("sleep 1000");  
            executor.shutdownNow();  
            System.out.println("end");  
        }  
    } 

``` 
- 输出
- begin
- sleep 1000
- end
- java.lang.InterruptedException: 循环被中断了
- at com.chinaso.phl.InterruptedTest$1.run(InterruptedTest.java:18)
- at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
- at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
- at java.lang.Thread.run(Thread.java:744)
- java.lang.InterruptedException: sleep interrupted
- at java.lang.Thread.sleep(Native Method) 


**中断状态标记**

中断机制通过使用称为中断状态的内部标记来实现。调用Thread.interrupt()设置这个标记。当线程通过调用静态方法Thread.interrupted()检测中断时，中断状态会被清除。非静态的isInterrupted()方法被线程用来检测其他线程的中断状态，不改变中断状态标记。

按照惯例，任何通过抛出一个InterruptedException异常退出的方法，当抛该异常时会清除中断状态。不过，通过其他的线程调用interrupt()方法，中断状态总是有可能会立即被重新设置。

Join()方法可以让一个线程等待另一个线程执行完成。若t是一个正在执行的Thread对象，


将会使当前线程暂停执行并等待t执行完成。重载的join()方法可以让开发者自定义等待周期。然而，和sleep()方法一样join()方法依赖于操作系统的时间处理机制，你不能假定join()方法将会精确的等待你所定义的时长。

如同sleep()方法，join()方法响应中断并在中断时抛出InterruptedException。

该例子中有2个线程，根据线程的名字判断是否需要等待，当然也可以设置其他的条件.

``` 
    package com.chinaso.phl;  
      
    public class JoinTest {  
      
        public static void main(String[] args) throws Exception {  
            MyThread t1 = new MyThread("t1");  
            MyThread t2 = new MyThread("t2");  
              
            t1.setThread(t2);  
            t2.setThread(t1);  
              
            t1.start();  
            t2.start();  
        }  
    }  
      
    class MyThread extends Thread {  
        private Thread thread;  
      
        public MyThread(String name) {  
            super(name);  
        }  
      
        @Override  
        public void run() {  
            for (int i = 0; i < 3; i++) {  
                try {  
                    // 如果当前线程是t2，则暂停让其他线程完成执行  
                    if (Thread.currentThread().getName().equals("t2")) {  
                        thread.join();  
                    }  
                    Thread.sleep(500);  
                    System.out.println("current thread is :" + this.getName() + ", thread.isAlive()="  + thread.isAlive());  
                } catch (Exception e) {  
                }  
            }  
        }  
      
        public Thread getThread() {  
            return thread;  
        }  
      
        public void setThread(Thread thread) {  
            this.thread = thread;  
        }  
    }  

``` 

- 输出结果
- current thread is :t1, thread.isAlive()=true
- current thread is :t1, thread.isAlive()=true
- current thread is :t1, thread.isAlive()=true
- current thread is :t2, thread.isAlive()=false
- current thread is :t2, thread.isAlive()=false
- current thread is :t2, thread.isAlive()=false 