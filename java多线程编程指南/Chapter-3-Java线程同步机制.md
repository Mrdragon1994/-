---
title: Chapter 3  Java线程同步机制
date: 2020-03-29 15:22:22
categories:
- 多线程
- Java
- Java多线程编程实战指南 核心篇 黄文海
tags:
- 线程
- Java
---

# Chapter 3  Java线程同步机制

## 3.1 线程同步机制简介

​	1.**线程同步机制** 是一套用于协调线程间的数据访问及活动的机制，该机制用于*保障线程安全*以及实现这些线程的共同目标。

​	2.从广义上将，Java平台提供的线程同步机制包括***锁***，***volatile***，***final***，***static***以及一些相关的***API***，如：***Object.wait()/Object.notify()***。

## 3.2 锁概述

1. 线程不安全的前提是多个线程并发访问共享数据，因此，我们容易想到应对这种多个线程同时访问共享数据的措施就是讲多个线程对共享数据的并发访问改为串行访问，即一个共享数据同一时刻只能被一个线程访问，该线程访问结束后其他线程才能对其进行访问，**锁(Lock)**就是利用这种思路进行线程安全的保证的。

2. 一个线程在访问共享数据前必须申请相应的锁，线程的这个动作被称为**锁的获得(acquire)**，一个线程获得某个锁，我们就称该线程为相应锁的持有线程，***一个锁一次只能被一个线程持有***。锁的持有线程可以对该锁保护的共享数据进行访问，访问结束后该线程必须**释放(release)**锁。锁的持有线程在其获得锁之后和释放锁之前的这段时间内锁操作的代码被称为**临界区(Critical Section)**。因此：==共享数据只允许在临界区被访问，且临界区一次只能被一个线程执行==。

3. 锁具有***排他性***，即一个锁一次只能被一个线程所持有，因此这种所被称为**排它锁**或**互斥锁**。后续还会提到一种**读写锁**，它是==排他锁==的一种改进。

   ![image-20200329152723893](Chapter-3-Java线程同步机制.assets/image-20200329152723893.png)

4. 按照JVM对锁的==**实现方式划分**==，Java平台中的锁包括**内部锁(Instrinsic Lock)**和**显式锁(Explicit Lock)**。内部锁是通过**synchronized**关键字来实现的，显式锁是通过**Lock**(如：java.concurrent.locks.ReentrantLock类)的实现类实现的。

   ### 3.2.1 锁的作用

   1. 锁能够保护共享数据以实现线程安全，其作用包括保障原子性，可见性和有序性；
   2. 锁是通过**互斥来保障原子性的**，因此一个锁一次只能由一个线程持有，那么该线程访问临界区时是不会受到其他线程操作共享数据的干扰的，这就使得临界区代码所执行的操作自然而然具备不可分割的特性，即具备了原子性；从互斥的角度看，锁其实就是将多个线程对共享数据的访问由并发转变为串行；
   3. 可见性的保障是通过写线程冲刷处理器缓存和读线程刷新处理器缓存这两个动作实现的，在Java平台中，锁的获得隐含着刷新处理器缓存这个动作，这使得读线程在执行临界区代码前（获得锁之后）可以将写线程对共享变量所做的更新同步到该线程执行处理器的高速缓存中，而锁的释放隐含着冲刷处理器高速缓存这个动作，这使得写线程对共享变量所做的更改能够被刷新到该线程执行处理器的告诉缓存中；因此，**锁能够保证可见性**。
   4. 锁能够保障有序性，写线程在临界区中锁执行的一系列操作在读线程锁执行的临界区看起来像是完全按照源代码顺序执行的。尽管锁能够保障有序性，但是这并不意味着临界区内的内存操作不会被重排序，临界区内的任意两个操作依然可以在临界区之内被重排序，**只是不会被重排序到临界区之外**，由于临界区内的操作是具有原子性的，写线程在临界区内对各个共享数据的更新同时对读线程可见，因此这种重排序并不会影响其他线程；
   5. 需要注意的是锁对原子性，可见性和有序性的保障是有2个必须同时满足的条件：
      - 这些线程在访问同一组共享数据的时候必须用的是同一个锁；
      - 这些线程中的任意一个线程，即使其仅仅是读取这组共享数据而没有对其进行更新的话，也需要在读取时持有相应的锁。
   
   ### 3.2.2 与锁有关的几个概念
   
   1. **可重入性**
   
      ***可重入性(Reentrancy)***描述的是一个线程在其持有一个锁的时候能否再次(或多次)申请该锁，如果可以申请，那么**该锁**就是可重入的，否则就是**非可重入的**：
   
      ```java
      void methodA() {
          acquireLock(lokc);  //申请锁lock
          methodB();  //调用方法B
          releaseLock(lock);  //释放锁lock
      }
      
      void methodB() {
          acquireLock(lock); //申请锁lock
          releaseLock(lock); //释放锁lock
      }
      ```
   
      ​	方法methodA中使用了锁lock，该锁引导的临界区中又调用了methodB，而方法B也需要使用锁lock，即该线程在methodA中已经获得了该锁的时候在methodB中仍旧需要获得该锁，可重入性便是描述这样一个问题。
   
      - 可重入锁可以被理解为一个对象，该对象包含一个计数器属性，计数器属性的初始值为0，表示相应的锁还没有被任何线程持有，每次线程获得一个可重入锁的时候，该锁的计数器值会被增加1，每次一个线程释放锁的时候，该锁的计数器属性值就会减少1。
   
   2. 锁的争用与调度
   
      Java平台中的锁的==**调度策略**==包括公平策略和非公平策略，相应的锁就被称为公平锁和非公平锁，**内部锁属于非公平锁，而显示锁既支持公平锁也支持非公平锁**。

## 3.3 内部锁：synchronized关键字

1. Java平台中的任何一个**对象**都有唯一一个与之关联的锁，这种所被称为**监视器(Monitor)**或者**内部锁(Intrinsic Lock)**，***内部锁是一种排他锁，内部锁属于非公平锁，内部锁可以保障原子性，可见性和有序性***；

2. 内部锁是通过synchronized关键字修饰的，synchronized关键字可以用来修饰方法和代码块；synchronized关键字修饰的方法被称为**同步方法**(Synchronized Method)，其中：修饰的静态方法称为**同步静态方法**，修饰的实例方法称为**同步实例方法**。同步方法的**整个方法体**就是一个**临界区**。

3. synchronized修饰的代码块被称为**同步块(Synchronized Block)**，

   ```java
   synchronized(锁句柄) {
   	//再次访问共享数据
   }
   ```

   ​	synchronized关键字所引导的代码块就是临界区，**锁句柄是一个对象的引用或能够返回对象的表达式**，例如，锁句柄可以填写为this关键字(表示当前对象)，作为锁句柄的变量通常用final修饰，这是因为锁句柄的值一旦改变，会导致执行同一个同步块的多个线程实际上使用的不是同一个锁，从而对共享数据产生竞态，因此我们常用***private final***修改锁句柄：

   ```java
   private final Object lock = new Object();
   ```

4. 同步**静态方法**方法相当于是以当前类本身作为锁句柄的同步块：

   ```java
   public class Demo {
   	public static synchronized void staticMethod() {
   		//临界区
   	} 
   }
   
   //相当于：
   public class Demo {
   	public static void staticMethod() {
   		synchronized {
   			//临界区
   		}
   	}
   }
   ```

   

5. 需要额外注意的是：JVM对同步块和同步方法的处理方式是不同的；

6. 内部锁的调度：首先，**JVM会为每个内部锁分配一个入口集(Entry Set)，用于存放申请锁失败的线程。**多个线程申请同一个锁的时候，只有一个申请者能够成为该锁的持有线程，而其他申请者的申请操作会失败，这些申请失败的线程不会抛出异常，而是会被暂停（生命周期状态变为BLOCKED状态）并被存入到**相应锁**的入口集中。当这些线程申请的锁被其持有线程释放后，该锁的入口集中的一个任意线程会被JVM唤醒，从而得到再次申请锁的机会，但是JVM对内部锁的调度仅支持非公平调度，被唤醒的等待线程占用处理器运行时还可能有其他新的活跃线程（RUNNABLE状态，且没有进入到入口集中）与该线程争抢锁，**因此被唤醒的线程不一定能够成为该锁的持有线程。**

## 3.4 显示锁：Lock 接口

1. 显示锁是JDK1.5开始引入的**排他锁**。作为一种线程同步机制，其作用和内部锁相同，但是提供了一些内部锁不具备的特性，但它不是内部锁的替代品;

2. 显示锁是java.util.concurrent.locks.Lock接口的实例，该接口对显示锁进行了抽象，类ReentrantLocl是Lock接口的默认实现类；该抽象类定义的基本方法如下：

   | 返回值    | 方法                                                         |
   | --------- | ------------------------------------------------------------ |
   | void      | lock() 获取锁                                                |
   | void      | lockInterruptibly() 如果当前线程未被中断，则获取锁           |
   | Condition | newCondition() 返回绑定到此Lock实例的新Condition实例         |
   | boolean   | tryLock() 仅在调用时锁为空闲状态才获取该锁                   |
   | boolean   | tryLock(long time, TimeUnit unit) 如果锁在给定时间内空闲，并且当前线程未被中断，则获取该锁 |
   | void      | unlock() 释放锁                                              |

3. 一个Lock接口实例就是一个显示锁对象，Lock接口定义的lock()和unlock()分别用于申请和释放相应Lock实例表示的锁，其用法如下：

   ```java
   private final Lock lock = ... ; //创建一个Lock实例
   
   lock.lock();  //申请锁lock
   try {
       //临界区
   } finally {
       //需要注意的是，lock必须在finally里释放，不然会造成锁泄露
       lock.unlock(); //释放锁lock
   }
   ```

   ​	显示锁在使用时，需要注意以下几个方面：

   - 创建Lock接口的实例。如果没有特别要求，我们就可以创建Lock默认的实现类ReentrantLock的实例作为显式锁来使用，这是一个可重入锁;
   - 在访问共享数据前要申请相应的显示锁，即Lock.lock();
   - Lock.lock()调用与Lock.unlock()调用之间的代码区域称为临界区，不过，我们一般认为上述中的try代码为临界区，因此，对共享数据访问的代码都应放在try{}中；
   - 为了避免锁泄露，应当将Lock.unlock()代码放在finally中，保证锁肯定是可以被释放的。

   ### 3.4.1 显式锁的调度

   1. ReentrantLock既支持公平锁又支持非公平锁，他有一个构造函数如下：

      ```java
      ReentrantLock(boolean fair) //true表示公平锁，false表示非公平锁
      ```

   2. 公平锁的调度往往是增加了线程的暂停和唤醒，增加了上下文切换为代价的，因此，公平锁更适合于锁被持有的时间较长或者线程申请锁的平均间隔时间相对较长的情形，公平锁的开销比非公平锁的开销大很多，因此，显式锁默认使用非公平锁的调度策略；

   ### 3.4.2 显式锁与内部锁的比较

   1. 内部锁是基于代码块的锁，灵活性差；仅仅由一个关键字来维护，无法充分发挥面向对象的灵活性；而且锁的申请和释放只能在一个方法内进行；

      显式锁是基于对象的锁，支持在一个方法内申请，在另外一个方法里释放锁。

   2. 内部锁基于代码块的这个特点也使其具有一个优势：简单易用，且不会出现锁泄露；而显式锁容易被错用而导致锁泄露，因此我们要**将锁的释放放在finally中**；

   3. 如果内部锁的持有线程一直不释放这个锁，那么同步在该锁上的所有线程就会一直被暂停而无法继续进行任务，而显式锁则可以轻松避免这个问题；Lock接口定义的tryLock()方法，该方法的作用是尝试申请相应Lock实例表示的锁，如果相应的锁**未被其他线程持有，那么该方法会返回true，**表示其获得了相应的锁；否则，该方法并不会导致其执行线程被暂停而是返回**false，表明其没有获得锁**；

      ```java
      Lock lock = ...; //创建一个Lock实例
      if	(lock.tryLock()) { //试图获得锁
          try {
              //临界区
          } finally {
              lock.unlock(); //释放锁
          }
      } else {
          //执行其他操作
      }
      ```

      tryLock()的一个带方法签名的重载方法是：tryLock(long time, TimeUnit unit)，可以指定一段时间去争抢锁，没有抢到在执行其他代码。

   4. 在锁的调度方面，***内部锁仅支持非公平锁，而显式锁既支持公平锁，又支持非公平锁***。

      ```java
      package com.thread.chapter3
      
      import java.util.concurrent.locks.Lock
      import java.util.concurrent.locks.ReentrantLock
      
      /**
       * @Author ChangQilong
       * @Date 2020/3/29 22:27
       */
      class TestLock {
          private static final def lock = new ReentrantLock() //默认是非公平锁
          private static int sharedData = 0  //共享数据为int类型的0
      
          public static void main(String[] args) {
              def t = new Thread(new Runnable() {
                  @Override
                  void run() {
                      lock.lock()  //获取锁
                      try {
                          try {
                              sleep(1000 * 50)
                          } catch(Exception e) {
                              e.printStackTrace()
                          }
                          sharedData = 1
                      } finally {
                          lock.unlock()
                      }
                  }
              })
              t.setName("Thread-01")
              t.start() //启动子线程
              sleep(100) //主线程休眠100ms
              lock.lock()
              try {
                  printf("sharedData = %d", sharedData)
              } finally {
                  lock.unlock()
              }
          }
      }
      
      ```

      ​	当程序启动后，在任务管理器中查出pid，然后在cmd中执行jstack -l {pid} >> 123.txt

      截取其部分堆栈信息：

      ```java
      2020-03-29 22:39:24
      Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):
      
      "Thread-01" #13 prio=5 os_prio=0 tid=0x0000000020952000 nid=0x26a0 waiting on condition [0x000000002118e000]  
         java.lang.Thread.State: TIMED_WAITING (sleeping) //语句②
      	at java.lang.Thread.sleep(Native Method)
      	at org.codehaus.groovy.runtime.DefaultGroovyStaticMethods.sleepImpl(DefaultGroovyStaticMethods.java:133)
      	at org.codehaus.groovy.runtime.DefaultGroovyStaticMethods.sleep(DefaultGroovyStaticMethods.java:155)
      	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
      	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
      	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
      	at java.lang.reflect.Method.invoke(Method.java:498)
      	at org.codehaus.groovy.runtime.metaclass.ReflectionMetaMethod.invoke(ReflectionMetaMethod.java:54)
      	at org.codehaus.groovy.runtime.metaclass.NewStaticMetaMethod.invoke(NewStaticMetaMethod.java:53)
      	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:325)
      	at org.codehaus.groovy.runtime.callsite.PogoMetaMethodSite$PogoMetaMethodSiteNoUnwrap.invoke(PogoMetaMethodSite.java:233)
      	at org.codehaus.groovy.runtime.callsite.PogoMetaMethodSite.callCurrent(PogoMetaMethodSite.java:59)
      	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCallCurrent(CallSiteArray.java:52)
      	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callCurrent(AbstractCallSite.java:154)
      	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callCurrent(AbstractCallSite.java:166)
      	at com.thread.chapter3.TestLock$1.run(TestLock.groovy:21)
      	at java.lang.Thread.run(Thread.java:748)
        Locked ownable synchronizers:
      	- <0x000000076c9bca88> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)     //语句②   
             
      ...省略部分没用信息
      "main" #1 prio=5 os_prio=0 tid=0x00000000012ae800 nid=0x59a8 waiting on condition [0x000000000346f000]
         java.lang.Thread.State: WAITING (parking)
      	at sun.misc.Unsafe.park(Native Method)
      	- parking to wait for  <0x000000076c9bca88> (a java.util.concurrent.locks.ReentrantLock$NonfairSync) //语句③
      	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
      	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
      	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:870)
      	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1199)
      	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
      	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
      	at java_util_concurrent_locks_Lock$lock.call(Unknown Source)
      	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)
      	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:113)
      	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:117)
      	at com.thread.chapter3.TestLock.main(TestLock.groovy:34)       
      ```

        我们观察语句③，发现“main”线程需要获得标识为0x000000076c9bca88的显示锁而被parking(等待)，而这个锁被线程“Thread-1”锁持有，如语句②。

   5. 显式锁提供了一个API用来对锁的信息进行监控，而内部锁不支持；ReentrantLock中定义的方法isLocked()可以用来检测某个相应锁是否被某个线程持有，getQueueLength()可以用来检查相应锁的等待线程数量；

   ### 3.4.3 锁的选用（内部锁还是显式锁）

   1. 内部锁是简单易用，不易造成锁泄露；显式锁的功能强大，灵活性高；
   2. 默认情况下选用内部锁，当多线程持有一个锁的时间相对较长或者线程申请锁的平均时间间隔相对较长的情况下使用非公平锁就不太恰当了，可以改用公平锁(显式锁)。

   ### 3.4.4 改进型锁：读写锁

   1. 锁的排他性会导致多个线程无法在同一时刻对共享变量进行读取（只读取而不更新），这不利于提供系统的并发性；

   2. **读写锁()**是一种改进型的排他锁，也被称为共享/排他锁；读写锁允许多个线程同时读取共享变量，但是一次只允许一个线程对共享变量进行更新，任何线程读取共享变量的时候，其他线程都无法对共享变量进行更新；一个线程更新共享变量的时候，其他线程都无法读取访问该变量；

   3. 总的来说，读锁对于读线程来说起到保护其访问的共享变量在其访问期间不被修改的作用，并使多个线程可以同时读取这些变量从而提高了并发性；而写锁保障了写线程能够以独占的方式安全地更新共享变量；写线程对共享变量的更新对读线程时可见的；

   4. java.util.concurrent.locks.ReadWriteLock接口是读写锁的抽象，其默认实现类时java.util.concurrent.ReentrantReadWriteLock。ReadWriteLock定义了两个方法，readLock()和writeLock()，这两个方法分别返回该锁实例的读锁和写锁，其类型是Lock；需要注意的是，**读锁和写锁返回的锁是同一个锁实例；**

   5. 读写锁的使用方式：

      ```java
      package com.thread.chapter3
      
      import java.util.concurrent.locks.ReadWriteLock
      import java.util.concurrent.locks.ReentrantReadWriteLock
      
      /**
       * @Author ChangQilong
       * @Date 2020/3/29 23:29
       */
      class ReadWriteLockUsage {
          private static final ReadWriteLock readWriteLock = new ReentrantReadWriteLock() //创建一个读写锁的实例
          private final def readLock = readWriteLock.readLock()  //通过读写锁拿到该读写锁的读锁
          private final def writeLock = readWriteLock.writeLock() //通过读写锁拿到该读写锁的写锁
          
          void reader() {
              readLock.lock()
              try {
                  //在此须区域读取共享变量
              } finally {
                  readLock.unlock()  //在finally中释放读锁
              }
          }
          
          void writer() {
              writeLock.lock()
              try {
                  //在此区域更新共享变量
              } finally {
                  writeLock.unlock() //在finally中释放写锁
              }
          }
      }
      
      ```

      ​	在原子性，可见性和有序性的保障上，读写锁起到的作用和普通的排他锁是一样的，**写线程释放写锁起到的作用相当于一个线程释放一个普通排它锁；读线程获得读锁所起到的作用相当于一个线程获得一个普通排他锁**，读写锁更适合以下场景：

      - 只读操作比写（更新）操作频繁得多；
      - 读线程持有锁的时间相对较长。

   6. **ReentrantReadWriteLock所实现的读写锁是一个可重入锁，它支持锁的降级，即一个线程持有读写锁的写锁情况下可以继续获得相应的读锁**：

      ```groovy
      package com.thread.chapter3
      
      import java.util.concurrent.locks.ReadWriteLock
      import java.util.concurrent.locks.ReentrantReadWriteLock
      
      /**
       * @Author ChangQilong
       * @Date 2020/3/29 23:44
       */
      class ReadWriteLockDowngrade {
          private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock()
          private final def readLock = readWriteLock.readLock()
          private final def writeLock = readWriteLock.writeLock()
      
          void operationWithLockDowngrade() {
              boolean readlockAcquired = false  //先创建一个标记位：读锁是否获得
              writeLock.lock()  //申请写锁
              try {
                  //对共享数据进行更新
                  //当前线程在持有写锁的情况下申请获得读锁
                  readLock.lock()  //申请读锁
                  readlockAcquired = true  //更改标志位
              } finally {
                  writeLock.unlock() //释放写锁
              }
              if (readlockAcquired) {
                  try {
                      //读取共享变量进行操作
                  } finally {
                      readLock.unlock()
                  }
              } else {
                  //执行其他操作
              }
          }
      }
      
      ```

      ​	***锁的降级的反面就是锁的升级，就是一个线程获得读锁的情形下去申请读锁，但是ReentrantReadWriteLock并不支持锁的升级，读线程如果想要申请写锁，那么必须要先释放读锁，然后才能释放写锁。***

      ```groovy
      package com.thread.chapter3
      
      import java.util.concurrent.locks.ReadWriteLock
      import java.util.concurrent.locks.ReentrantReadWriteLock
      
      /**
       * ReentrantReadWriteUpgrade是不支持锁的升级的，该demo是为了验证不支持锁的升级
       * @Author ChangQilong
       * @Date 2020/3/30 0:01
       */
      class ReentrantReadWriteUpgrade {
          private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock()
          private final def readLock = readWriteLock.readLock()
          private final def writeLock = readWriteLock.writeLock()
          
          private static def data = 0
          public static void main(String[] args) {
              ReentrantReadWriteUpgrade readWriteUpgrade = new ReentrantReadWriteUpgrade()
              Thread t = new Thread(new Runnable() {
                  @Override
                  void run() {
                      readWriteUpgrade.operationWithLockToUpgrade()
                  }
              })
              
              t.start()
          }
          
          void operationWithLockToUpgrade() {
              boolean writeLockAcquired = false 
              readLock.lock()  //读锁的申请
              try {
                  println("${data}")
                  writeLock.lock() //写锁的申请
                  data = 1
                  writeLockAcquired = true
              } finally {
                  readLock.unlock() //释放读锁
              }
              if (writeLockAcquired) {
                  try {
                      println("${data}")
                  } finally {
                      writeLock.unlock()
                  }
              } else {
                  println("failed")
              } 
          }
      }
      
      ```

      ​	如上代码，如果不释放读锁而直接进行写锁的申请，会出现死锁的情况，线程被卡死，只能输出：

      ```shell
      0
      ```

      

      ```groovy
      package com.thread.chapter3
      
      import java.util.concurrent.locks.ReadWriteLock
      import java.util.concurrent.locks.ReentrantReadWriteLock
      
      /**
       * ReentrantReadWriteUpgrade是不支持锁的升级的，该demo是为了验证不支持锁的升级
       * @Author ChangQilong
       * @Date 2020/3/30 0:01
       */
      class ReentrantReadWriteUpgrade {
          private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock()
          private final def readLock = readWriteLock.readLock()
          private final def writeLock = readWriteLock.writeLock()
      
          private static def data = 0
          public static void main(String[] args) {
              ReentrantReadWriteUpgrade readWriteUpgrade = new ReentrantReadWriteUpgrade()
              Thread t = new Thread(new Runnable() {
                  @Override
                  void run() {
                      readWriteUpgrade.operationWithLockToUpgrade()
                  }
              })
      
              t.start()
          }
      
          void operationWithLockToUpgrade() {
              boolean writeLockAcquired = false
              readLock.lock()  //读锁的申请
              try {
                  println("${data}")
                 
              } finally {
                  readLock.unlock() //释放读锁 
                  writeLock.lock() //写锁的申请
                  data = 1
                  writeLockAcquired = true
              }
              if (writeLockAcquired) {
                  try {
                      println("${data}")
                  } finally {
                      writeLock.unlock()
                  }
              } else {
                  println("failed")
              }
          }
      }
      
      ```

      ​	如上，先释放读锁，再进行写锁的申请，是可以顺利执行的。最后输出：

      ```shell
      0
      1
      ```

   
## 3.5 锁的适用场景

​	锁是Java线程同步机制中功能最强大，适用范围最广，同时也是开销最大，可能导致的问题最多的同步机制。多个线程共享同一组数据的时候，如果有线程涉及到如下操作，就可以考虑使用锁的机制：

- check-then-act操作：一个线程读取共享数据并在此基础上决定下一步的走向；
  
   - read-modify-write操作：一个线程读取共享数据并在此基础上更新数据，不过，自增操作(i++)这种简单的read-modify-write操作，可以使用**原子变量**来实现线程安全；
   
- 多个线程对多个共享数据进行更新，如果这些共享数据之间存在关联关系，为了保障操作的原子性我们也可以考虑锁。

     

   ## 3.6 线程同步机制的底层助手：内存屏障

   1. [3.2.1](#3.2.1 锁的作用)节说到过，锁在保障可见性的时候是因为线程在获得锁和释放锁的时候回执行两个操作，获得锁的时候会刷新处理器缓存，释放锁的时候会冲刷处理器缓存。对于同一个锁保护的共享数据而言，前一个动作保证了该锁的持有线程能够读取到前一个持有线程对这些共享数据所做的更新，后一个动作保证了该锁的持有线程对这些共享数据所做的更新对该锁的后续持有线程可见。其实现原理见2；

   2. JVM是借助**内存屏障(Memory Barrier，又称Fence)**来实现上述两个动作的；内存屏障是对一类仅针对内存读，写操作指令的跨处理器架构的底层的抽象或称呼。**内存屏障是被插入到两个指令之间进行使用的，其作用是禁止编译器，处理器重排序从而保障有序性。**它在指令序列(如指令1；指令2；指令3)中就像是一堵墙(so we call it Fence~)，一样使其两侧(之前和之后)的指令无法“穿越”它(一旦穿越就是重排序了)。***为了实现禁止重排序的功能，这些指令也往往具有一个副作用--刷新处理器缓存和冲刷处理器缓存，从而保证了可见性***，

   3. **扩展**：通常所说的**读操作(Load或Read)**是指将主内存中的数据(通过高速缓存)读取到寄存器中，**写操作(Store或Write)**是指将数据写到共享内存中；内部锁的申请与释放对应的JVM字节码指令分别是MonitorEnter和MonitorExit；

   4. 按照内存屏障所起的作用来划分，将内存屏障划分为以下几种：

      - 按照可见性保障来划分：内存屏障可分为**加载屏障(Load Barrier)**和**存储屏障(Store Barrier)**。加载屏障的作用是**刷新处理器缓存**，存储屏障的作用是**冲刷处理器缓存**。JVM会在MonitorExit(释放锁)对应的机器码指令之后插入一个存储屏障，保障写线程在释放锁之前在临界区中对共享变量所做的更新对读线程的执行处理器来说是可同步的；同理，JVM会在MonitorEntry(申请锁)对应的机器码指令之后临界区开始之前插入一个加载屏障，这使得读线程的执行处理器能够将写线程对相应共享变量所做的更新从其他处理器同步到该处理器的告诉缓存中。**因此，可见性的保障是通过写线程和读线程成对地使用存储屏障和加载屏障实现的。**

      - 按照有序性保障来划分：内存屏障可以分为**获取屏障(Acquire Barrier)**和**释放屏障(Release Barrier)**。**获取屏障的使用方式是在一个读操作(包括read-modify-write及普通的读操作)之后插入该内存屏障**，其作用是禁止该读操作与其后的任何读写操作之间进行重排序，这相当于在进行后续操作之前先要获得相应共享数据的所有权。**释放屏障的使用方式是在一个写操作之前插入该内存屏障**，其作用是禁止该写操作与其前面的任何读写操作之间进行重排序，这相当于在对相应共享数据操作结束后释放所有权。JVM会在MonitorEnter(它包含了读操作)对应的机器码指令之后临界区之前的地方插入一个获取屏障，并在临界区结束之后MonitorExit(包含了写操作)对应的机器码指令之前的地方插入一个释放屏障。

        ![image-20200330110707426](Chapter-3-Java线程同步机制.assets/image-20200330110707426.png)


   5. 为了保障线程安全，我们需要使用Java线程同步机制，而内存屏障则是JVM在实现Java线程同步机制所用的具体“工具”。

## 3.7 锁与重排序

1. 为了使锁能够起到其预定的功能并且尽量避免对性能造成“影响”，编译器（JIT编译器）和处理器应当遵守一些重排序规则，这些重排序规则禁止一部分的重排并且允许另外一部分的重排；

2. 临界区的语句可以被(编译器)重排序到临界区之内，而临界区之内的操作无法被（编译器或处理器）重排到临界区之外；临界区内，临界区前和临界区后这3个区域内的任意两个操作都可以在各自个区域范围内进行重排序(只要相应的重排序能够满足貌似串行语义)；

   ![image-20200330144030342](Chapter-3-Java线程同步机制.assets/image-20200330144030342.png)



3. 规则1：临界区内的操作不允许被重排序到临界区之外--该规则是保障原子性和可见性的基础，编译器和处理器都必须遵守该规则，JVM虚拟机会在临界区的开始之前和结束之后分别插入一个获取屏障和释放屏障，从而禁止临界区内的操作被排到临界区之前和之后；

4. 规则2：临界区内的操作允许被重排序--因为锁的排他性会保证临界区内的操作是一个原子操作，而重排序有利于发挥处理器的计算性能；

5. 规则3：临界区外(临界区之前或临界区之后)的操作可以被重排序--该规则也是为了发挥处理器的计算性能；

   ---

   上面3条规则的时候，我们假定代码中只有一个临界区，但是实际情况下，可能存在多个临界区，因此为了保证上述规则得到满足且不出现死锁，以下规则也要被满足：

6. 规则4：锁申请(MonitorEntry)和锁释放(MonitorExit)操作不能被重排序--锁的申请与锁的释放总是配对出现的，即一个线程总是先成功申请(获得)锁然后才能释放锁；

7. 规则5：两个锁的申请操作不能被重排序；

8. 规则6：两个锁的释放不能被重排序；

   规则5和规则6确保了Java语义支持嵌套锁(即一个同步块中又包含了一个同步块)的使用，并且避免(申请，释放)可能导致的死锁，编译器和处理器都必须遵守这些规则。

9. 规则7：临界区外(临界区前和临界区后)的操作可以被重排序到临界区内。

## 3.8 轻量级同步机制：volatile关键字

1. volatile关键字用来修饰**共享可变变量**，即没有被**final**关键字修饰的实例变量或静态变量，相应的变量就被称为volatile变量；

   ```java
   private volatile int logLevel;
   ```

2. volatile关键字修饰的变量的值容易变动，因而不稳定；意味着对这种变量的读和写操作都必须从高速缓存或者主内存(也是通过高速缓存读取)中读取，以读取变量的相对新值；

3. volatile关键字常被称为轻量级锁，其作用与锁有相同的地方，保证可见性和有序性，但是不同的是：在**原子性**方面它既能保障volatile变量操作的原子性，没有锁的排他性；其次，volatile关键字的使用不会引起上下文切换(这是volatile被冠以“轻量级”的原因)；

   ### 3.8.1 volatile的作用

   1. volatile关键字的作用包括：保障可见性，有序性和保障long/double型读写操作的原子性；

   2. 一般而言，对volatile变量的赋值操作，其右边表达式只要涉及共享变量(包括被赋值的volatile变量本身)，那么这个赋值操作就不是原子操作。要保障这样操作的原子性，我们仍需要借助锁。

      ```java
      private volatile long count1 = 0L;
      count1 = count2 + 1; //操作①
      ```

      当count2是一个共享变量时，那么操作①就不是原子操作，如果count2是一个局部变量，那么该赋值操作就是一个原子操作。

   3. **volatile关键字在原子性方面仅保障对被修饰的变量的读、写操作本身的原子性。如果要保障对volatile变量的赋值操作的原子性，那么这个赋值操作不能涉及任何共享变量(包括被赋值的volatile变量本身)的访问。**

   4. 写线程对volatile变量的写操作会产生类似于释放锁的效果，读线程对volatile变量的读操作会产生类似于获得锁的效果，因此，volatile具有可见性和有序性；

   5. 对于volatile变量的写操作，JVM会在该操作之前插入一个释放屏障(保障有序性)，并在该操作之后插入一个存储屏障(保障可见性)

      ![image-20200331230600941](Chapter-3-Java线程同步机制.assets/image-20200331230600941.png)

      其中，释放屏障禁止了volatile写操作与该操作之前的任何读、写操作进行重排序，从而保证了volatile写操作之前的任何读、写操作会先于volatile写操作被提交；而存储屏障可以保障对volatile变量的写操作完成后，对其他线程是可见的。

   6. 对于volatile变量的读操作，JVM会在该操作之前插入一个加载屏障(保证可见性)，在该操作会后插入一个获取屏障(保障有序性)

      ![image-20200402152842020](Chapter-3-Java线程同步机制.assets/image-20200402152842020.png)

      ​	其中，加载屏障通过冲刷处理器缓存，使其执行线程所在的额处理器将其他处理器对共享变量所做的更新操作能够同步到该处理器的高速缓存中，保障了可见性；获取屏障禁止了volatile读操作之后的任何读、写操作与volatile读操作进行重排序，因此它保障了有序性。
      
   7. 如果被修饰的变量是个数组，那么volatile关键字只能够对数组引用本身的操作(读取数组引用和更新数组引用)起作用，而无法对数组元素的操作(读取，更新数组元素)；如果想要对数组元素的读、写操作也能够触发volatile关键字的作用，那么我们可以使用AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray；

   ### 3.8.2 volatile变量的开销

   ​		  volatile变量的开销包括读变量和写变量两个方面。volatile变量的读、写操作都不会导致上下文切换，因此volatile的开销要比锁小。

   ### 3.8.3 volatile的典型应用场景

   1. volatile用于保障long/double型变量的读、写操作的原子性；
   2. 使用volatile变量作为状态标志；应用程序的某个状态由一个线程设置，其他线程会读取该状态并以改状态作为其计算依据；
   3. 使用volatile保障可见性，多个线程共享一个可变状态变量，当其中一个线程更新了该变量之后，其他线程在无需加锁的情况下也能够看到该更新；
   4. 多个线程共享**一组可变状态变量**时，通常我们需要使用锁来保障对这些变量的更新操作的原子性，以避免产生数据不一致问题，利用**volatile变量写操作具有的原子性，我们可以把这一组可变状态变量封装成一个对象，然后对这些可变状态变量的更新操作就可以通过创建一个新的对象并将该对象的引用赋值给相应的引用型变量来实现**，在这个过程中，volatile保障了原子性和可见性，从而避免了锁的使用；

## 3.9 正确实现看似简单的单例模式

1. 

   ```java
   1. package com.thread.chapter3
   
   /**
   
    * 本Demo采用双重检测机制实现的单例模式
   
    * @Author ChangQilong* @Date 2020/4/2 16:27
      */
      class DCLSingleton {
   
      /**
   
       * 保存该类的唯一实例，通过volatile保障可见性和有序性
         */
         private static volatile DCLSingleton instance
   
      /**
       *
   
       * 将构造器私有化，保证无法通过new创建该类的实例
         */
         private DCLSingleton() {
   
      }
   
      static DCLSingleton getInstance() {
          if (null == instance) {
              synchronized (DCLSingleton.class) {
                  if (null == instance) {
                      instance = new DCLSingleton()
                  }
              }
          }
          return instance
      }
      }
   ```

2.

```java
package com.thread.chapter3

/**
 * 静态内部类实现单例模式
 * @Author ChangQilong* @Date 2020/4/2 16:33
 */
class StaticInnerClassSingleton {

    public static void main(String[] args) {
        StaticInnerClassSingleton.getInstance().doSomething()
    }

    //私有化的构造器
    private StaticInnerClassSingleton() {

    }

    //静态内部类
    private static class InstanceHolder {
        final static StaticInnerClassSingleton INSTANCE = new StaticInnerClassSingleton()
    }

    static StaticInnerClassSingleton getInstance() {
        return InstanceHolder.INSTANCE
    }

    void doSomething() {
        //省略部分代码
    }
}

```

3. 

   ```java
   package com.thread.chapter3
   
   import groovy.util.logging.Slf4j
   
   /**
    * 使用枚举实现单例模式
    * 枚举类型Singleton相当于一个单例类，其字段INSTANCE值相当于该类的唯一实例，这个实例在Singleton.INSTANCE初次被引用的时候
    * 才会被初始化，仅访问Singleton本身(比如Singleton.class.getName())是不会导致Singleton的唯一实例被初始化的
    * @Author ChangQilong* @Date 2020/4/2 16:36
    */
   @Slf4j
   class EnumBasedSingleton {
       public static void main(String[] args) {
           log.info(Singleton.class.getName())
           Singleton.INSTANCE.doSomething()
       }
   }
   
   enum Singleton {
       INSTANCE;
       //私有构造器
       private Singleton() {
       }
   
       void doSomething() {
   
       }
   }
   ```



## 3.10 CAS与原子变量

### 3.10.1 CAS

1. CAS(Compare and Swap)是对一种处理器指令的称呼，不少多线程相关的Java标准类库的实现最终都会借助于CAS；

2. 如下代码：

   ```java
   public void increment() {
   	synchronized(this) {
   		count++
   	}
   }
   ```

   在这里我们使用锁保障原子性，但是锁的开销是很大的，另外，volatile虽然开销小一些，但是它无法保障“count++”这样自增操作的原子性，事实上，保障像自增操作这种比较简单的原子性有更好的选择--CAS，CAS能够将read-modify-write和check-and-act之类的操作转变为原子操作；

3. CAS的伪代码如下：

   ```java
   do {
       oldValue = V.get();
       newValue = calculate(oldValue);
   } while ( !compareAndSwap(V, oldValue, newValue))
       
   boolean compareAndSwap(Varible V, Object A, Object B) {
       if (A == V.get()) {
           V.set(B);
           return true;
       }
       return false;
   }    
   ```

   在循环体中读取贡献变量V的旧值A，并以该值作为输入经过一系列操作计算共享变量的新值B，接着调用CAS试图将V的值更新为B，若更新失败，则会继续重试，直到成功；

   **CAS只能保证共享变量更新的原子性，并不能保障可见性；因此，我们需要将共享变量用volatile来修饰。**

### 3.10.2 原子操作工具：原子变量类

1. 原子变量类(Atomics)是基于CAS实现的能够保障对共享变量进行read-modify-write更新操作的原子性和可见性的一组工具类；可以被看作是增强型的volatile变量，原子变量类一共有12个，可以被分为4组；

   | 分组         | 类                                                           |
   | ------------ | ------------------------------------------------------------ |
   | 基础数据类型 | AtomicInteger,AtomicLong,AtomicBoolean                       |
   | 数组型       | AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray      |
   | 字段更新器   | AtomicIntegerFieldUpdater,AtomicLongFieldUpdater,AtomicReferenceFieldUpdater |
   | 饮用型       | AtomicReference,AtomicStampedReference,AtomicMarkableReference |

   CAS的ABA问题

   对于共享变量V，当前线程看到它的值为A的那一刻，其他线程已经将其值更新为了B，接着当前线程执行CA的时候该变量的值又被更新为A，这便出现了ABA的问题，解决该问题的思路是：为共享变量的更新引入一个修订号，每次更新共享变量时相应的修订号加1。

   

   

   ## 3.11 对象的发布与逸出

   1. 对象发布(Publish)是指使对象能够被其作用域之外的线程访问；

   2. 常见的几种对象发布形式：

      - private修饰共享变量

        ```java
        private Map<String, Integer> register = new HashMap<String, Integer>();
        ```

      - public修饰共享变量

        ```java
        public Map<String, Integer> register = new 
        HashMap<String, Integer>();
        ```
        
      
      ​               
      
      - 在非private方法(包括public,protected,package方法)中返回一个对象
      
        ```java
        private Map<String, Integer> register = new HashMap<String, Integer>();
        public Map<String, Integer> getRegister() {
            return this.register;
        }
        ```
      
      - 创建内部类，使得当前对象(this)能够被这个内部类使用
      
        ```java
        public void startTask(final Object task) {
            Thread t = new Thread(new Runnable(){
                @Override
                public void run() {
                    
                }
            }); 
        }
        ```
      
        上述代码中“new Runnable()”所创建的匿名内部类可以访问其外层类的当前实例this，（通过外层类名.this）
      
      - 通过方法调用将对象传递给外部方法
      
      当我们需要发布一个对象的时候就需要注意与之相关的线程安全问题，当然，我们可以斟酌使用锁，volatile关键字来保障线程安全，下面还有其他能够保证线程安全的问题；
      
      ### 3.11.1 对象的初始化安全：重访final与static
      
      1. static关键字在多线程环境下有特殊的含义，它能保障一个线程即使在未使用其他同步机制的情况下也总是可以读取到一个类的静态变量的初始值；但是，这种可见性保障仅限于线程初次读取该变量值，如果这个静态变量在相应的类初始化完毕之后被其他线程更新过，那么一个线程要读取该变量的相对新值仍需借助锁，volatile关键字；
      
      2. 如下代码，init()方法所启动的线程至少可以看到语句①~语句③觉的操作结果，即该线程总是可以看到static字段taskConfig的初始值，如果init()方法被执行的时候(甚至在此之前)其他线程执行了changeConfig方法，那么init方法中启动的线程能够读取到taskConfig的相对新增也是没有保障的；
      
         ```java
         public class StaticVisibilityExample {
             private static Map<String, String> taskConfig;
             static {
                 taskConfig = new HashMap<String, String>(); //语句①
                 taskConfig.put("url", "https://12"); //语句②
                 taskConfig.put("timeout","1000"); //语句③    
             }
             
             public static void changeConfig(String url, int timeout) {
                 taskConfig = new HashMap<String, String>(); //语句④
                 taskConfig.put("url", url); //语句⑤
                 taskConfig.put("timeout", String.valueOf(timeout)); //语句⑥
             }
             
             public static void init() {
                 //该线程至少能够看到语句①到语句③的操作结果，是否能看到语句④~语句⑥的操作是没有保障的
                 Thread t = new Thread(() ->
                                       {
                                           doTask(taskConfig.get("url"), Integer.valueOf(taskConfig.get("timeout")))
                                       })
             }
         }
         ```
      
         通过上述代码可以得出结论：**static关键字仅仅保障读线程能够读到相应字段的初始值，而不是相对新增。**
      
      3. **对于引用型静态变量，static关键字还能够保障一个线程读取到该变量的初始值时，这个值所指向(引用)的对象已经初始化完毕；**
      
      4. final关键字在多线程环境下有特殊作用，当一个对象被发布到其他线程的时候，该对象的所有final字段都是初始化完毕的，即其他线程读取这些字段的时候所读取到的值都是相应字段的初始值(而不是默认值)，而非final字段没有这种保障，即这些线程读取该对象的非final字段时读取到的值可能仍旧是相应字段的默认值。**对于引用型final字段，final关键字还进一步确保该字段所引用的对象已经初始化完毕，即这些线程读取该字段所引用的对象的各个字段时所读取到的值都是相应字段的初始值。**
   
   ### 3.11.2 安全发布与逸出
   
   1. **安全发布**就是指对象以一种线程安全的方式被发布，当一个对象的发布出现我们不期望的结果或者对象发布本身不是我们所期望的时候，我们就称该**对象逸出**，这是一种不安全的发布；
   2. 常见的几种对象逸出的方式：
      - 在构造器中将this赋值给一个共享变量；
      - 在构造器中将this作为方法参数传递给其他方法；
      - 在构造器中启动基于匿名类的线程。
















































































































