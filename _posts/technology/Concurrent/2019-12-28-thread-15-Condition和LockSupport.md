---
layout: post
title: Java 并发编程15--Condition和LockSupport
date:  2019/12/28 15:58:15 
categories: document
tag: java并发编程

---

* content
{:toc}



# 1.Condition #

## 1.1.Condition介绍 ##


Condition的作用是对锁进行更精确的控制。Condition中的await()方法相当于Object的wait()方法，Condition中的signal()方法相当于Object的notify()方法，Condition中的signalAll()相当于Object的notifyAll()方法。不同的是，Object中的wait(),notify(),notifyAll()方法是和"同步锁"(synchronized关键字)捆绑使用的；而Condition是需要与"互斥锁"/"共享锁"捆绑使用的。

## 1.2.Condition函数列表 ##

```
// 造成当前线程在接到信号或被中断之前一直处于等待状态。
void await()
// 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
boolean await(long time, TimeUnit unit)
// 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
long awaitNanos(long nanosTimeout)
// 造成当前线程在接到信号之前一直处于等待状态。
void awaitUninterruptibly()
// 造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。
boolean awaitUntil(Date deadline)
// 唤醒一个等待线程。
void signal()
// 唤醒所有等待线程。
void signalAll()
```

## 1.3.Condition示例 ##

示例1是通过Object的wait(), notify()来演示线程的休眠/唤醒功能。
示例2是通过Condition的await(), signal()来演示线程的休眠/唤醒功能。
示例3是通过Condition的高级功能。

**示例1**
```
public class WaitTest1 {

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        synchronized(ta) { // 通过synchronized(ta)获取“对象ta的同步锁”
            try {
                System.out.println(Thread.currentThread().getName()+" start ta");
                ta.start();

                System.out.println(Thread.currentThread().getName()+" block");
                ta.wait();    // 等待

                System.out.println(Thread.currentThread().getName()+" continue");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            synchronized (this) { // 通过synchronized(this)获取“当前对象的同步锁”
                System.out.println(Thread.currentThread().getName()+" wakup others");
                notify();    // 唤醒“当前对象上的等待线程”
            }
        }
    }
}

```

**示例2：**
```
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionTest1 {
        
    private static Lock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        lock.lock(); // 获取锁
        try {
            System.out.println(Thread.currentThread().getName()+" start ta");
            ta.start();

            System.out.println(Thread.currentThread().getName()+" block");
            condition.await();    // 等待

            System.out.println(Thread.currentThread().getName()+" continue");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();    // 释放锁
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            lock.lock();    // 获取锁
            try {
                System.out.println(Thread.currentThread().getName()+" wakup others");
                condition.signal();    // 唤醒“condition所在锁上的其它线程”
            } finally {
                lock.unlock();    // 释放锁
            }
        }
    }
}

```

运行结果
```
main start ta
main block
ta wakup others
main continue

```

通过“示例1”和“示例2”，我们知道Condition和Object的方法有一下对应关系：

```
状态         Object      Condition  
休眠          wait        await
唤醒个线程     notify      signal
唤醒所有线程   notifyAll   signalAll


```
划重点了。。。


Condition除了支持上面的功能之外，它更强大的地方在于：能够更加精细的控制多线程的休眠与唤醒。对于同一个锁，我们可以创建多个Condition，在不同的情况下使用不同的Condition。


例如，假如多线程读/写同一个缓冲区：当向缓冲区中写入数据之后，唤醒"读线程"；当从缓冲区读出数据之后，唤醒"写线程"；并且当缓冲区满的时候，"写线程"需要等待；当缓冲区为空时，"读线程"需要等待。         


如果采用Object类中的wait(), notify(), notifyAll()实现该缓冲区，当向缓冲区写入数据之后需要唤醒"读线程"时，不可能通过notify()或notifyAll()明确的指定唤醒"读线程"，而只能通过notifyAll唤醒所有线程(但是notifyAll无法区分唤醒的线程是读线程，还是写线程)。  但是，通过Condition，就能明确的指定唤醒读线程。


看看下面的**示例3**，可能对这个概念有更深刻的理解。

```
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer {
    final Lock lock = new ReentrantLock();
    final Condition notFull  = lock.newCondition(); 
    final Condition notEmpty = lock.newCondition(); 

    final Object[] items = new Object[5];
    int putptr, takeptr, count;

    public void put(Object x) throws InterruptedException {
        lock.lock();    //获取锁
        try {
            // 如果“缓冲已满”，则等待；直到“缓冲”不是满的，才将x添加到缓冲中。
            while (count == items.length)
                notFull.await();
            // 将x添加到缓冲中
            items[putptr] = x; 
            // 将“put统计数putptr+1”；如果“缓冲已满”，则设putptr为0。
            if (++putptr == items.length) putptr = 0;
            // 将“缓冲”数量+1
            ++count;
            // 唤醒take线程，因为take线程通过notEmpty.await()等待
            notEmpty.signal();

            // 打印写入的数据
            System.out.println(Thread.currentThread().getName() + " put  "+ (Integer)x);
        } finally {
            lock.unlock();    // 释放锁
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();    //获取锁
        try {
            // 如果“缓冲为空”，则等待；直到“缓冲”不为空，才将x从缓冲中取出。
            while (count == 0) 
                notEmpty.await();
            // 将x从缓冲中取出
            Object x = items[takeptr]; 
            // 将“take统计数takeptr+1”；如果“缓冲为空”，则设takeptr为0。
            if (++takeptr == items.length) takeptr = 0;
            // 将“缓冲”数量-1
            --count;
            // 唤醒put线程，因为put线程通过notFull.await()等待
            notFull.signal();

            // 打印取出的数据
            System.out.println(Thread.currentThread().getName() + " take "+ (Integer)x);
            return x;
        } finally {
            lock.unlock();    // 释放锁
        }
    } 
}

public class ConditionTest2 {
    private static BoundedBuffer bb = new BoundedBuffer();

    public static void main(String[] args) {
        // 启动10个“写线程”，向BoundedBuffer中不断的写数据(写入0-9)；
        // 启动10个“读线程”，从BoundedBuffer中不断的读数据。
        for (int i=0; i<10; i++) {
            new PutThread("p"+i, i).start();
            new TakeThread("t"+i).start();
        }
    }

    static class PutThread extends Thread {
        private int num;
        public PutThread(String name, int num) {
            super(name);
            this.num = num;
        }
        public void run() {
            try {
                Thread.sleep(1);    // 线程休眠1ms
                bb.put(num);        // 向BoundedBuffer中写入数据
            } catch (InterruptedException e) {
            }
        }
    }

    static class TakeThread extends Thread {
        public TakeThread(String name) {
            super(name);
        }
        public void run() {
            try {
                Thread.sleep(10);                    // 线程休眠1ms
                Integer num = (Integer)bb.take();    // 从BoundedBuffer中取出数据
            } catch (InterruptedException e) {
            }
        }
    }
}

```

**(某一次)运行结果：**

```
p1 put  1
p4 put  4
p5 put  5
p0 put  0
p2 put  2
t0 take 1
p3 put  3
t1 take 4
p6 put  6
t2 take 5
p7 put  7
t3 take 0
p8 put  8
t4 take 2
p9 put  9
t5 take 3
t6 take 6
t7 take 7
t8 take 8
t9 take 9
```

**结果说明：**

(01) BoundedBuffer 是容量为5的缓冲，缓冲中存储的是Object对象，支持多线程的读/写缓冲。多个线程操作“一个BoundedBuffer对象”时，它们通过互斥锁lock对缓冲区items进行互斥访问；而且同一个BoundedBuffer对象下的全部线程共用“notFull”和“notEmpty”这两个Condition。


notFull用于控制写缓冲，notEmpty用于控制读缓冲。当缓冲已满的时候，调用put的线程会执行notFull.await()进行等待；当缓冲区不是满的状态时，就将对象添加到缓冲区并将缓冲区的容量count+1，最后，调用notEmpty.signal()缓冲notEmpty上的等待线程(调用notEmpty.await的线程)。 简言之，notFull控制“缓冲区的写入”，当往缓冲区写入数据之后会唤醒notEmpty上的等待线程。 


同理，notEmpty控制“缓冲区的读取”，当读取了缓冲区数据之后会唤醒notFull上的等待线程。


(02) 在ConditionTest2的main函数中，启动10个“写线程”，向BoundedBuffer中不断的写数据(写入0-9)；同时，也启动10个“读线程”，从BoundedBuffer中不断的读数据。


(03) 简单分析一下运行结果。
```
	 1, p1线程向缓冲中写入1。    此时，缓冲区数据:   | 1 |   |   |   |   |
     2, p4线程向缓冲中写入4。    此时，缓冲区数据:   | 1 | 4 |   |   |   |
     3, p5线程向缓冲中写入5。    此时，缓冲区数据:   | 1 | 4 | 5 |   |   |
     4, p0线程向缓冲中写入0。    此时，缓冲区数据:   | 1 | 4 | 5 | 0 |   |
     5, p2线程向缓冲中写入2。    此时，缓冲区数据:   | 1 | 4 | 5 | 0 | 2 |
     此时，缓冲区容量为5；缓冲区已满！如果此时，还有“写线程”想往缓冲中写入数据，会调用put中的notFull.await()等待，直接缓冲区非满状态，才能继续运行。
     6, t0线程从缓冲中取出数据1。此时，缓冲区数据:   |   | 4 | 5 | 0 | 2 |
     7, p3线程向缓冲中写入3。    此时，缓冲区数据:   | 3 | 4 | 5 | 0 | 2 |
     8, t1线程从缓冲中取出数据4。此时，缓冲区数据:   | 3 |   | 5 | 0 | 2 |
     9, p6线程向缓冲中写入6。    此时，缓冲区数据:   | 3 | 6 | 5 | 0 | 2 |
     ...
```

# 2.LockSupport #

## 2.1LockSupport介绍 ##

LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程，而且park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。

因为park() 和 unpark()有许可的存在；调用 park() 的线程和另一个试图将其 unpark() 的线程之间的竞争将保持活性。


## 2.2LockSupport函数列表 ##
```
// 返回提供给最近一次尚未解除阻塞的 park 方法调用的 blocker 对象，如果该调用不受阻塞，则返回 null。
static Object getBlocker(Thread t)
// 为了线程调度，禁用当前线程，除非许可可用。
static void park()
// 为了线程调度，在许可可用之前禁用当前线程。
static void park(Object blocker)
// 为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。
static void parkNanos(long nanos)
// 为了线程调度，在许可可用前禁用当前线程，并最多等待指定的等待时间。
static void parkNanos(Object blocker, long nanos)
// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
static void parkUntil(long deadline)
// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
static void parkUntil(Object blocker, long deadline)
// 如果给定线程的许可尚不可用，则使其可用。
static void unpark(Thread thread)

```


## 2.3LockSupport示例 ##

对比下面的“示例1”和“示例2”可以更清晰的了解LockSupport的用法。

**示例1：**
```
public class WaitTest1 {

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");

        synchronized(ta) { // 通过synchronized(ta)获取“对象ta的同步锁”
            try {
                System.out.println(Thread.currentThread().getName()+" start ta");
                ta.start();

                System.out.println(Thread.currentThread().getName()+" block");
                // 主线程等待
                ta.wait();

                System.out.println(Thread.currentThread().getName()+" continue");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            synchronized (this) { // 通过synchronized(this)获取“当前对象的同步锁”
                System.out.println(Thread.currentThread().getName()+" wakup others");
                notify();    // 唤醒“当前对象上的等待线程”
            }
        }
    }
}
```

**示例2：**
```
import java.util.concurrent.locks.LockSupport;

public class LockSupportTest1 {

    private static Thread mainThread;

    public static void main(String[] args) {

        ThreadA ta = new ThreadA("ta");
        // 获取主线程
        mainThread = Thread.currentThread();

        System.out.println(Thread.currentThread().getName()+" start ta");
        ta.start();

        System.out.println(Thread.currentThread().getName()+" block");
        // 主线程阻塞
        LockSupport.park(mainThread);

        System.out.println(Thread.currentThread().getName()+" continue");
    }

    static class ThreadA extends Thread{

        public ThreadA(String name) {
            super(name);
        }

        public void run() {
            System.out.println(Thread.currentThread().getName()+" wakup others");
            // 唤醒“主线程”
            LockSupport.unpark(mainThread);
        }
    }
}
```

**运行结果：**

```
main start ta
main block
ta wakup others
main continue

```

**说明：**park和wait的区别。wait让线程阻塞前，必须通过synchronized获取同步锁。