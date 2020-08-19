---
title: Chapter 5 线程间的协作
date: 2020-04-21 22:44:27
categories:
- 多线程
- Java
- Java多线程编程实战指南 核心篇 黄文海
tags:
- 线程
- Java
---

# Chapter 5 线程间协作

## 5.1 等待与通知：wait/notify

1. 在单线程中，如果执行目标操作需要满足一定条件时才能执行，那么我们可以使用if语句来进行判断，但是在多线程中，由于判断条件可能只是暂时不满足，带共享变量更新后，该条件就满足了，因此我们可以使用如下伪代码：

   ```java
   atomic { //原子操作
   	while(保护条件不成立) {
           暂停当前线程;
       }
       //执行目标动作
       doAction();
   }
   ```

   显然,上述操作必须具有原子性，这里，一个线程因其执行了目标动作所需的保护条件未满足而被暂停的过成被称为**等待(Wait)**,一个线程更新了系统状态，使得其他线程所需的保护条件得到满足的时候唤醒那些被暂停的线程的过成就被称为**通知(Notify)**。

### 5.1.1 wait/notify的作用和用法

1. Object.wait()/wait(long time)用来实现等待;Object.notify()/notifyAll()用来实现通知；

2. wait()的作用是使其执行线程被暂停(生命周期状态变为WAINTING),该方法的执行线程被称为**等待线程**；notify()的作用是使其执行的线程被唤醒,该方法的执行线程被称为**通知线程**；

3. 由于Object类时Java中任何对象的父类，因此使用Java中的任何对象都能够实现等待与通知；

4. ***由于一个线程只有在持有一个对象的内部锁的情况下才能够调用该对象的wait方法，因此Object.wait()调用总是放在相应对象所引导的临界区之中；***

   ```java
   synchronized(someObject) { //获得某个对象的内部锁
   	//只有在获得某个对象的内部锁后，才能够调用该对象身上的wait()
       while(保护条件不成立) {
           someObject.wait();
       }
       //代码执行到这里说明保护条件已经满足
       daAction();
   }
   ```

5. 包含上述模板代码的方法被称为**受保护方法**，受保护方法包括3个要素：保护条件，暂停当前线程和执行目标动作；这是因为等待线程对保护条件的判断及目标动作的执行必须是原子操作，否则可能产生竞态，因此目标动作的执行必须和保护条件的判断以及Object.wait()调用放在同一个对象所引导的临界区中；

6. someObject.wait()会以原子操作的方式使其执行线程(当前线程)暂停并使该线程释放其持有的someObject对应的内部锁（**但此时wait()方法并未返回**）,因此someObject.wait()(同一个对象的同一个方法)可以被多个线程执行，因此一个对象身上可能存在多个等待线程；

7. someObject.notify()可以唤醒someObject上的一个(任意)等待线程；**被唤醒的等待线程在其占用处理器继续运行的时候，需要再次申请someObject身上对应的内部锁**；被唤醒的线程当其获得someObject身上的锁后，会继续执行someObject.wait()中剩余的指令，直到wait方法返回；

8. **重要**：

   - 等待线程对保护条件的判断、Object.wait()的调用总是应该放在相应对象所引导的临界区中的一个循环语句之中；
   - 等待线程对保护条件的判断、Object.wait()的执行以及目标动作的执行必须放在用一个对象(内部锁)锁引导的临界区之中；
   - Object.wait()暂停当前线程时释放的锁只是与该wait方法所属对象的内部锁，当前线程所持有的其他内部锁，显示锁并不会因此而被释放。

9. 使用Object.notify()实现通知的伪代码如下：

   ```java
   synchronized(someObject) {
       //更新等待线程的保护条件所涉及的共享变量
       updateSharedState();
       //唤醒其他线程
       someObject.notify();
   }
   ```

10. 包含上述模板代码的方法称为**通知方法**，它包含两个要素：更新共享变量、唤醒其他线程；

11. 同理，一个线程只有在持有一个对象的内部锁的情况下才能够执行该对象的notify方法，因此Object.notify()调用总是放在相应对象内部锁所引导的临界区中，也正是由于Object.notify()要求其执行线程必须持有该方法所属对象的内部锁，因此Object.wait()在暂停其执行线程的同时，也必须释放相应的内部锁；否则通知线程无法获得相应的内部锁；也就无法执行相应对象的notify方法来通知等待线程；

12. Object.notify()的执行线程**持有的相应对象的内部锁**只有在Object.notify()调用所在的**临界区代码执行结束后才会被释放**，而不同于wait(),notify()本身不会释放内部锁；因此，为了使等待线程在其被唤醒后能够尽快获得内部锁，因此我们要将notify()调用尽量放在临界区的最后；

13. **重要**：

    - 等待线程和通知线程必须调用同一个对象的wait方法,notify方法来实现等待和通知；
    - 调用一个对象的notify方法锁唤醒的线程仅仅是该对象上的一个任意线程，因此可以使用notifyAll()方法；
    - notify方法调用应该尽可能放在临界区的结束地方。

14. wait/notify的内部实现：**JVM会为每个对象维护一个入口集(Entry Set)用于存储申请该对象内部锁的线程；此外，JVM还会为每个对象维护一个被称为等待集(Wait Set)的队列，该队列用于存储该对象上的等待线程。Object.wait()将当前线程暂停并释放相应内部锁的同时会将当前线程(的引用)存入该方法所属对象的等待集中。执行一个对象的notify方法会使该对象的等待集中的一个任意线程被唤醒。被唤醒的线程仍旧会停留在相应对象的等待集中，直到该线程再次持有相应的内部锁的时候(此时，Object.wait()调用尚未结束)Object.wait()会使当前线程从其所在的等待集中移除，接着Object.wait()方法就返回了。Object().wait()/notify()实现的等待/通知中的几个关键动作，包括将当前线程加入等待集，暂停当前线程，释放锁移交将唤醒后的线程从等待集中移除，都是在Object.wait()中实现的。**

### 5.1.2 wait/notify的开销及问题

1. 过早唤醒(Wakeup too soon)：设一组等待/通知线程同步在对象someObject上，初始状态下所有的保护条件均不成立，接着，线程N1更新了共享变量state1使得保护条件成立，此时为了唤醒使用该保护条件的所有等待线程(线程W1和W2)，N1执行了someObject.notifyAll()，但是W2所使用的保护条件2此时并没有成立，这就将使得该线程被唤醒后仍旧需要继续等待，这种等待线程在其所需的保护条件并未成立的情况下被唤醒的现象就被称为**过早唤醒**，从而导致资源浪费。

   - 解决该问题可以使用**JDK1.5引入的java.util.concurrent.locks.Condition接口**来解决。

   ![image-20200427213032681](Chapter-5-线程间的协作.assets/image-20200427213032681.png)

2. 信号丢失(Missed Signal):

   - 一个表现是：等待线程在执行Object.wait()前没有先判断保护条件是否已经成立，那么可能会出现以下情形--当通知线程在该等待线程进入临界区之前已经更新了共享变量，使得相应的保护条件成立并进行了通知，但是此时等待线程还没有被暂停，自然也就无所谓唤醒了；这就可能导致等待线程直接执行Object.wait()而被暂停的时候，该线程由于没有其他线程进行通知而一直处于等待状态；

     - 解决该问题**只要将保护条件的判断和Object.wait()调用放在一个循环语句之中就可以避免信号丢失的问题**

   - 另一个表现是：对于使用同一个保护条件的多个等待线程，如果只是通知线程调用了Object.notify()，那么最多会唤醒该对象身上的一个等待线程，设置一个都不会唤醒，因为可能唤醒的是该对象上使用其他保护方法的一个等待线程；但是notify()在其唤醒线程时不考虑任何保护条件，因此可能导致其他等待线程都没有机会被唤醒

     - 解决该问题**在必要时使用notifyAll()，而不是notify()**
   
3. 欺骗性唤醒(Spurious Wakeup):等待线程在没有其他线程唤醒的情况下被唤醒，但此时共享变量可能仍旧处于不满足条件的情形，这种问题是操作系统的问题

   - 解决方法是将共享变量的保护条件和Object.wait()放在同一个临界区中的循环语句中。

4. 上下文切换：

   - 首先，等待线程执行Object.wait()至少会导致该线程对相应对象内部锁的两次申请和释放；通知线程在执行notify()/notifyAll()需要持有相应对象的内部锁；而这种锁的申请会导致上下文切换；

   - 其次，等待线程从被暂停到唤醒这个过程本身就会导致上下文切换；

   - 再次，被唤醒的线程在继续运行时会和相应对象入口集中的其他线程以及其他新来的活跃线程争用相应的内部锁，导致上下文切换；

   - 最后，过早唤醒也会导致上下文切换，这是因为被过早唤醒的线程仍然需要等待，即再次经历被暂停和唤醒的过程。

     解决上下文切换问题：

     - 在保证程序正确的情况下，尽量使用notify()而不是notifyAll();因为nofityAll()会导致过早唤醒；
     - 通知线程在执行完notify()/notifyAll()后应尽快释放锁(就是尽量安排这两个方法的调用放在临界区的后面),这样可以避免被唤醒的线程在Object.wait()调用返回前再次申请相应的内部锁，由于该锁尚未被通知线程释放而导致该线程被暂停



### 5.1.3 Object.notify()/notifyAll()的选用

1. Object.notify()可能导致信号丢失，而notifyAll又会导致过早唤醒从而占用资源；
2. **优先使用notifyAll()确保正确性，只有在有证据表明使用Object.notify()足够的情况下才是用Object.notify()**，notify()只有在下列条件***全部满足***的条件下才可取代notifyAll:
   - Condition 1: 一次通知至多只需要唤醒一个线程，但是光这点还不能保证使用notify()的正确性，因为不同的等待线程可能使用不同的保护条件，而notify()唤醒的是该对象身上的任意一个等待线程，可能唤醒的不是我们改变保护条件的那个等待线程；
   - Condition 2: 等待线程使用的是同一个保护条件，并且这些线程在Object.wait()后执行的处理逻辑是一样的。典型的场景是：使用同一个Runnable接口实例创建不同的线程或者从同一个Thread子类new出多个实例。



### 5.1.4 wait()/notify()与Thread.jion()

1. Thread.join()可以使当前线程等待目标线程结束后才继续运行，Thread.join(long timemillis)允许我们指定一个时间，如果目标线程没有在指定的时间内终止，那么当前线程也会继续执行。

2. Thread.join()底层是用wait()和notify()实现的。

3. Thread.join(0)等同于Thread.join()

4. 比如线程A中加入了线程B.join方法，则线程A默认执行wait方法，释放资源进入等待状态，此时线程B获得资源，执行结束后释放资源，线程A重新获取自CPU，继续执行。由此实现线程的顺序执行。

   ```java
   package com.thread.chapter5
   
   /**
    * @Author ChangQilong
    * @Date 2020/4/27 23:17
    */
   class UsingJoin {
       public static void main(String[] args) {
           Thread thread3 = new ThirdThread()
           thread3.start()
       }
   }
   
   class FirstThread extends Thread {
       @Override
       public void run() {
           println("1")
       }
   }
   
   class SecondThread extends Thread {
       @Override
       public void run() {
           Thread thread = new FirstThread()
           thread.start()
           thread.join()
           println("2")
       }
   }
   
   class ThirdThread extends Thread {
       @Override
       public void run() {
           Thread thread = new SecondThread()
           thread.start()
           thread.join()
           println("3")
       }
   }
   ```



## 5.2 Java条件变量

1. JDK1.5引入了Condition接口可以作为wait()/notify()的替代品来实现等待/通知；它为解决过早唤醒问题提供了支持，并解决了wait(long)不能区分其返回时等待超时还是被通知线程唤醒的问题，Condition接口提供了await(),signal()和signalAll()方法相当于是wait(),notify()和notifyAll()。
2. 

 

 

 





