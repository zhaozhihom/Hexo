---
title: wait()和notify()方法的注释解读
date: 2017-04-29 20:21:30
categories:
tags:
    - Java
---

今天闲来看了看Object类的源码，发现注释里都是大神们满满的爱，比如wait()、notify()、notifyAll()的注释里详细的描述了方法的作用，用法，注意事项，甚至还列出了推荐书单。JDK开发人员写的注释，才是真正的一手资料吧哈哈。

<!--more-->

## 源码

#### wait(long timeout)

```Java
/* This method causes the current thread (call it <var>T</var>) to
     * place itself in the wait set for this object and then to relinquish
     * any and all synchronization claims on this object. Thread <var>T</var>
     * becomes disabled for thread scheduling purposes and lies dormant
     * until one of four things happens:
     * <ul>                                                     
     * <li>Some other thread invokes the {@code notify} method for this              
     * object and thread <var>T</var> happens to be arbitrarily chosen as
     * the thread to be awakened.
     * <li>Some other thread invokes the {@code notifyAll} method for this      
     * object.
     * <li>Some other thread {@linkplain Thread#interrupt() interrupts}            
     * thread <var>T</var>.
     * <li>The specified amount of real time has elapsed, more or less.  If     
     * {@code timeout} is zero, however, then real time is not taken into
     * consideration and the thread simply waits until notified.
     * </ul>
     
     * <p>
     * A thread can also wake up without being notified, interrupted, or        
     * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
     * occur in practice, applications must guard against it by testing for
     * the condition that should have caused the thread to be awakened, and
     * continuing to wait if the condition is not satisfied.  In other words,
     * waits should always occur in loops, like this one:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * (For more information on this topic, see Section 3.2.3 in Doug Lea's
     * "Concurrent Programming in Java (Second Edition)" (Addison-Wesley,
     * 2000), or Item 50 in Joshua Bloch's "Effective Java Programming      
     * Language Guide" (Addison-Wesley, 2001).
     **/
public final native void wait(long timeout) throws InterruptedException;
    
```
- 翻译一下：
                        
        此方法导致当前同步的线程进入等待队列，并且释放锁，以下四种情况可以把线程移出等待队列：
        1. 其它线程调用notify()方法，刚好唤醒了它
        2. 其它线程调用了notifyAll()方法
        3. 其它线程调用Thread的interrupt()方法打断了休眠
        4. 到达wait(long time)设定的时间
        还有一种特殊情况，线程会因不知名的原因被唤醒，称为虚假唤醒，而且这种现象已经在实际环境中证实，因此，wait()一般要写在循环中使用，像这样：
      
         synchronized (obj) {
              while (<condition does not hold>)
                  obj.wait(timeout);
              ... // Perform action appropriate to condition
          }

        如果要更清楚地搞懂这个问题，推荐两本书，Addison-Wesley出版的《Java并发编程》，Joshua Bloch编写的《Effective Java》第50条。
        
- 编写JDK的前辈们已经把路都修好了，还在等什么，快把书从箱底里拿出来。

#### notify()

```Java
 /**
     * Wakes up a single thread that is waiting on this object's
     * monitor. If any threads are waiting on this object, one of them
     * is chosen to be awakened. The choice is arbitrary and occurs at
     * the discretion of the implementation. A thread waits on an object's
     * monitor by calling one of the {@code wait} methods.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. A thread becomes the owner of the
     * object's monitor in one of three ways:                       
     * <ul>
     * <li>By executing a synchronized instance method of that object.  
     * <li>By executing the body of a {@code synchronized} statement    
     *     that synchronizes on the object.
     * <li>For objects of type {@code Class,} by executing a                      
     *     synchronized static method of that class.
     * </ul>
     * <p>
     * Only one thread at a time can own an object's monitor.
     */
    public final native void notify();
```
- 翻译：
        
        此方法随机唤醒等待队列中的一条线程，这个方法只能被占有锁的线程使用
        获得对象锁的几种方式：
        1. 执行一个对象的同步实例方法
        2. 利用同步代码块同步一个对象
        3. 对于类对象，执行类中的一个静态同步方法
        每次只能有一条线程占有对象的锁

## 测试
上面两个方法的注释中，已经说明了关于锁的基本性质，我写了一些测试，来证明这些方法的用法。

- 测试 wait()，notify()
```Java

public class TestWaitNotify {

    public static void main(String[] args) { //                myThread          main
    
        MyThread myThread = new MyThread("myThread"); 
                                //创建线程myThread             新建态           运行态
        
        synchronized (myThread){    //main获取 myThread对象的监听器(锁)
            try{
            
                System.out.println(Thread.currentThread().getName()+" start myThread");
                myThread.start();   //启动myThread             就绪态

                System.out.println(Thread.currentThread().getName()+" wait");
                myThread.wait(); //main主线程释放锁            运行态           阻塞态

                System.out.println(Thread.currentThread().getName()+" continue");

            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}

class  MyThread extends Thread{

    public  MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        synchronized (this){
            System.out.println(Thread.currentThread().getName()+" get the monitor");
            notify();                   //myThread释放锁 解除main的阻塞状态                 
            System.out.println(Thread.currentThread().getName()+" over");
        }                                        //             死亡态         就绪态
    }

}
    /*运行结果
    main start myThread
    main wait
    myThread get the monitor
    myThread over
    main continue
    */

```

- wait(long time)测试，这个测试使两个线程每隔三秒交替运行
```Java

public class TestWait {

    public static void main(String[] args) {                 
        MyThread1 myThread = new MyThread1("myThread"); 
        synchronized (myThread){                           
            try{
                myThread.start();                           

                System.out.println(Thread.currentThread().getName()+" run");
                while(true) {       //《Effective Java》中建议wait()方法在循环中使用
                    myThread.wait(6000);
                    System.out.println(Thread.currentThread().getName() + " run");
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }

    }
}

class  MyThread1 extends Thread{

    public  MyThread1(String name) {
        super(name);
    }

    @Override
    public void run() {
        synchronized (this){
            try {
                wait(3000);
                System.out.println(Thread.currentThread().getName()+ " run");
                while(true) {
                    wait(6000);
                    System.out.println(Thread.currentThread().getName() + " run");
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }

}

    /* 运行结果
    main run
    myThread run
    main run
    myThread run
    main run
    myThread run
    main run
    myThread run
    main run
    myThread run
    main run
    myThread run
    main run

    ····
    */
```
- notifyAll()测试，这个测试证明了等待队列是一个类似于栈的数据结构，遵循后进先出原则
```Java

public class TestNotifyAll {

    static Object obj = new Object();

    public static void main(String[] args) {

        for(int i = 0; i < 5; i++){  //              自建线程                      main
            new MyThread(" "+i).start(); //         新建 就绪                      运行
        }

        try {
            System.out.println(Thread.currentThread().getName()+" sleep(3000)");
                                            //       运行 阻塞                   休眠
            Thread.sleep(3000);             // 为其它线程留运行时间
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        synchronized(obj) {  //                      阻塞                       获取锁 运行
            // 主线程等待唤醒。
            System.out.println(Thread.currentThread().getName()+" notifyAll()");
            obj.notifyAll();      //             唤醒 运行                      释放锁 死亡
        }
    }

    static class MyThread extends Thread{

        public MyThread(String name) {
            super(name);
        }

        @Override
        public void run() {
            synchronized (obj) {      //                      获取锁
                try {
                    // 打印输出结果
                    System.out.println(Thread.currentThread().getName() + " wait");
                    // 当前线程阻塞 释放锁资源
                    obj.wait();
                    // 打印输出结果
                    System.out.println(Thread.currentThread().getName() + " run");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
    
    /* 运行结果
     main sleep(3000)
    1 wait
    0 wait
    2 wait
    4 wait
    3 wait
    main notifyAll()
    3 run
    4 run
    2 run
    0 run
    1 run
    */ 
     
```

- 读源码和注释是个学习的好方法，路还很长，需要一点点积累






