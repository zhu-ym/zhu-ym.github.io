---
title: Java并发基础
typora-root-url: ../
date: 2021-03-19 09:34:13
tags:
- 线程
- 并发
categories:
- [Java并发]
- [Java语法,进阶]
---

介绍一下Java线程的基础知识，了解如何在多线程条件下保证线程安全

<!--more-->

# 一、使用线程

### 实现Runnable接口

Runnable接口可以作为参数传入到Thread构造器中初始化生成一个线程，所以可以通过实现Runnable接口使用线程。

Runnable是函数式接口，只含一个声明方法run，当一个线程被调度时就会执行该线程的run方法，可以直接用lambda表达式实现

```java
package achieveThread;

public class UseRunnable {
	public static void main(String[] args) {
		Runnable r1 = ()->{
			System.out.println(Thread.currentThread().getName()+" begin:"+System.currentTimeMillis());
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName()+" end:"+System.currentTimeMillis());
		};
		Thread t1 = new Thread(r1);
		t1.setName("test");
		t1.start();
		Thread t2 = new Thread(r1);
		t2.start();	
	}
}
```

### 继承Thread类

Thread类也实现了Runnable接口，也需要重写run方法

```java
package achieveThread;

public class UseThread extends Thread{
	public void run() {
		System.out.println(Thread.currentThread().getName()+" begin:"+System.currentTimeMillis());
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName()+" end:"+System.currentTimeMillis());
	}
	public static void main(String[] args) {
		UseThread t = new UseThread();
		t.start();
	}
}
```



**实现接口和继承Thread类的对比**

- 实现接口可以把并行任务和运行机制解耦

- 继承整个类可能开销要大

- Java不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口

  

# 二、线程状态

线程有下图六种状态、可以用getState方法确定当前状态

![image-20210319094804228](/images/image-20210319094804228.png)

- new即新建状态：创建后尚未启动。

- runnable即可运行状态：主要包含两种情况ready（就绪）和running(运行中)

  - 调用了start方法只是进入就绪状态
  - 就绪状态的线程在获得CPU时间片后变为运行中状态

- blocked即阻塞状态：当一个线程试图获取内部的对象锁（synchronized），该锁被其他线程所占有，就会进入阻塞状态，直到允许持有这个锁时，才会回到可运行状态

- waiting即等待状态：当线程等待另一个线程通知调度器出现一个条件时，该线程就会进入等待状态。比如Object.wait、Thread.join、或者等待Java.util.concurrent库中Lock、Condition时就会进入等待状态。

  Object类的notify和notifyAll 方法, 会让线程从Waiting 转化为Runnable 状态，Thread.join方法会等待join的线程执行完毕, 才会把线程状态变成Runnable 状态.

- timed waiting即计时等待状态：调用带有超时方法的方法。要么符合条件被唤醒，要么等待到设定时间被唤醒

- teminated即终止状态：正常执行或者意外中断退出

# 三、多线程方案

大多数并发程序是围绕任务执行构建的，当有很多任务需要执行时，我们可能需要使用多线程

这时一般有两种方案

- **显式的为任务创建线程**

  即为每一个任务分配一个线程，期待提高吞吐率

  但是在一定范围内，增加线程可以提高系统吞吐率，但是超出这个范围后，过多的线程只会降低程序的执行速度。此外线程的生命周期开销非常高，过多的线程可能会导致资源耗竭。

  这时，提出了另一种方案

- **在线程池中执行任务**

  线程池是管理一组同构工作线程的资源池，其关键在于重用线程池的工作线程而不是创建新线程，从而分摊了线程创建销毁过程中的巨大开销

  除此之外，当任务到来时，通常工作线程已经存在，因此不会由于等待创建线程而延迟任务的执行，从而提高了响应性。

  最后，通过适当调整线程池的大小，可以创建足够多的线程以便使处理器保持忙碌状态，又可以防止过多线程相互竞争资源而使应用程序耗竭内存、资源失败

# 四、线程池

## Executor框架

Java中线程池的创建主要有两种，都是基于Executor框架，首先了解一下Executor框架

![image-20210319103302128](/images/image-20210319103302128.png)

其中Executor接口定义了execute()方法，可以接收Runnable任务

`ExecutorService`接口继承了Executor接口，是一个比Executor使用更广泛的子类接口。其定义了submit()方法可以提交Runnable或者Callable任务，并且返回一个Future对象，从而可以利用这个Future对象描述任务。

`ThreadPoolExecutor`类是我们生成线程池的主要方式之一,返回的线程池都实现了`ExecutorService`接口

最后还有一个工具类`Executors`类，其很多工厂方法可以生成线程池，其底层原理还是调用了`ThreadPoolExecutor`类的构造方法实现

综上了解线程池的产生，主要还是得了解`ThreadPoolExecutor`类



### 利用ThreadPoolExecutor生成线程池

- **生成**

  下面是类`ThreadPoolExecutor`通用构造方法

  ```java
  public ThreadPoolExecutor(int corePoolSize,
                                int maximumPoolSize,
                                long keepAliveTime,
                                TimeUnit unit,
                                BlockingQueue<Runnable> workQueue,
                                ThreadFactory threadFactory,
                                RejectedExecutionHandler handler)
  ```

  其中：

  - corePoolSize表示线程池大小，核心池大小，即基本大小
  - maximumPoolSize表示线程池最大线程数
  - keepAliveTime:当线程数量大于corePoolSize值，若空闲线程超过此时间单位则回收
  - unit：keepAliveTime的时间单位
  - workQueue：执行前用于保持任务的队列，有三种：无界队列、有界队列、同步移交
  - threadFactory：可以定义自己的Thread线程
  - handler：饱和策略

  当任务来临后，下图可以体现这些参数的作用

  ![image-20210319105224288](/images/image-20210319105224288.png)

下面讲具体讲解这些参数

- **核心池大小的选择**

  我们需要对线程池的大小进行适当调整，当线程池中核心线程数量过大时，线程与线程之间会争取CPU资源，这样就会导致上下文切换。过多的上下文切换会增加线程的执行时间，影响了整体执行的效率；当线程池中的核心线程数量过少时，如果同一时间有大量任务需要处理，可能会导致大量任务在任务队列中排队等待执行，甚至会出现队列满了之后任务无法执行的情况，或者大量任务堆积在任务队列导致内存溢出（OOM）

  - 对于CPU密集型任务，由于CPU密集型任务的性质，导致CPU的使用率很高，如果线程池中的核心线程数量过多，会增加上下文切换的次数，带来额外的开销。因此，一般情况下线程池的核心线程数量等于CPU核心数+1。

  - 对于I/O密集型任务，由于I/O密集型任务CPU使用率并不是很高，可以让CPU在等待I/O操作的时去处理别的任务，充分利用CPU。因此，一般情况下线程的核心线程数等于2*CPU核心数。

- **线程工厂**

  线程工厂即是自己定义线程池中的Thread类，其主要靠实现ThreadFactory接口实现，而接口中，只声明了newThread(Runnable r)方法代表自己生成自定义的线程

  ```java
  public interface ThreadFactory {
      Thread newThread(Runnable r);
  }
  ```

  通常我们要自己定义一个Thread类，主要可以用来定制一些线程名、自己处理异常等

- **拒绝策略**

  当线程池中资源全部被占用时（工作队列满、线程池达最大容量），对新添加的Task任务有不同处理策略，也叫饱和策略

  - AbortPolicy：任务 被拒绝时，抛出RejectExecutionException异常、为默认
  - CallerRunsPolicy：被拒绝时，会调用当前线程池的所在的线程去执行被拒绝的任务，即在主线程执行，阻塞主线程（优点后面提交的任务会等待，缺点也是阻塞）
  - DiscardOldestPolicy：被拒绝时，线程池放弃等待队列中最旧的任务添加它
  - DiscardPolicy：被拒绝时，丢弃它

### 利用Executors工厂类生成线程池

- **使用newCachedThreadPool创建可缓存线程**

  ```java
  public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
      
  public static ExecutorService newCachedThreadPool()    
  ```

  其自动回收空闲线程和添加新线程，规模不受限制

  默认使用SynchronousQueue，避免直接排队，直接提交任务

- **利用newFixedThreadPool创建有界线程池**

  ```java
   public static ExecutorService newFixedThreadPool(int nThreads)
   public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
  ```

  int型参数代表线程池最大数量，每提交一个任务创建一个线程，直到达到最大数量，空闲线程保留

  默认采用无界队列LinkedBlockingQueue

- **利用newSingleThreadExecutor创建单一线程池**

  ```java
  public static ExecutorService newSingleThreadExecutor()
  public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) 
  ```

  创建单一线程池，能保证按照任务在队列中的顺序串行执行

  默认采用无界队列LinkedBlockingQueue

- **利用newScheduledThreadPool创建可延时执行的有界线程池**

  参数同newFixedThreadPool一致，可以延迟、定时执行

# 五、多线程如何保证线程安全

## 线程安全性

线程安全：当多个线程访问一个类时，不管这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步和协同，这个类都能表现出正确的行为，那么称这个类是线程安全的。

但如果如果多个线程共享一个可变的状态变量没有使用合适的同步，程序就会出现错误，影响线程安全性。

比如常见的：

- 竞态条件：某个计算的正确性取决于多个线程交替执行时序：常见有Check-Then-Act操作、读取-修改-写入操作
- 数据竞争：访问共享的非final域没有采用同步协同

## 1.不可变

多使用不可变对象，不可变对象一定是线程安全的。

常见不可变类型：

- String、Integer、Long等
- 枚举类型
- final关键词修饰的基本数据类型
- 利用`Collections.unmodifiable...()`获取的不可变集合（视图）

## 2.互斥同步

利用一般用synchronized或ReentrantLock实现同步实现互斥同步，同步块在已进入时会阻塞其他线程。

## 3.非阻塞同步

基于冲突检测的乐观并发策略：先操作，如果没有其他线程征用直接成功，有再进行补偿，这种实现常常不用将线程挂起，也叫非阻塞同步。

原理：CAS

许多原子类应用了这个方案

## 4.线程封闭

当某个对象被封闭在一个线程中，将自动实现线程安全性

**1.栈封闭**

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```java
public class StackClosedExample {
    public void add100() {
        int count = 0;
        for (int i = 0; i < 100; i++) {
            count++;
        }
        System.out.println(count);
    }
}
```

上诉代码中的count是局部变量，在在这个方法中无论多少个线程调用，输出总为100

**2.ThreadLocal类**

ThreadLocal类可以使得线程中的某个值和保存值的对象关联起来，并且为每个线程都有独立的一份值副本。通过get和set等访问接口或方法获取值。

显然，当某个频繁执行的操作需要某个对象，而同时又希望避免在每次执行时都重新分配临时变量，就可以使用这项技术

# 六、互斥同步

## 1.Synchronized

- **原理**

  Jvm基于进入和退出Monitor对象来实现方法同步和代码块同步。

  其中：

  - 方法级的同步是隐式，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。JVM可以从方法常量池中的方法表结构 中的 `ACC_SYNCHRONIZED `访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会 检查方法的` ACC_SYNCHRONIZED `访问标志是否被设置，如果设置了，执行线程将先持有monitor（虚拟机规范中用的是管程一词）， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor。

  - 代码块的同步是利用`monitorenter`和`monitorexit`这两个字节码指令。它们分别位于同步代码块的开始和结束位置。当JVM执行到`monitorenter`指令时，当前线程试图获取monitor对象的所有权，如果未加锁或者已经被当前线程所持有，就把锁的计数器+1；当执行`monitorexit`指令时，锁计数器-1；当锁计数器为0时，该锁就被释放了。如果获取monitor对象失败，该线程则会进入阻塞状态，直到其他线程释放锁。
    

- **使用**
  - 通常作用形式：

  ```java
  synchronized(obj){
    // 代码块...
  }
  ```

  synchronized只作用于一个对象，当一个线程持有这个对象的锁时，另一个线程再访问这个对象的同步代码块就会进入阻塞（JVM可以优化为自旋）

  - 同步一个方法

    ```java
    public synchronized void func () {
        // ...
    }
    ```

  当同步一个方法时，实际是作用于持有这个方法的对象

  - 同步一个类

    ```java
    public void func() {
        synchronized (A.class) {
            // ...
        }
    }
    ```

  当同步一个类时，实际是作用于这个类，即两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步

  - 同步一个静态方法

    ```java
    public synchronized static void func () {
        // ...
    }
    ```

  即同步了这个类

- **条件队列**

  Java中线程之间的任务存在状态依赖，即在这些任务中有一些操作有着基于状态的前提条件，比如不能从空队列中删除一个元素、或获取一个尚未结束任务的结果，

  依赖状态的操作可用一直堵塞到可以继续执行，在synchronized中可以通过内置条件队列实现这个操作。

  条件队列是指可以使得一组线程能够通过某种方式来等待特定的条件为真，元素是线程的队列。如同一个对象可以作为一个锁一般，一个对象也可以作为一个条件队列。

  在条件等待中，包括加锁、wait方法和一个条件谓语，在条件谓语中包含多个状态变量，这些状态由一个锁来保护，要求测试条件谓语时先持有这个锁，又要求锁对象和条件队列对象必须是同一个对象。

  通常一个内置条件队列的使用包括等待与通知，通过一个条件谓语判断是否将一个线程放入这个对象的条件队列中，又通过另一个线程改变了状态将条件队列中的线程唤醒

  - **等待**

    下面是状态依赖方法的标准形式

    ```java
    void stateDependentMethod() throws InterruptedException{
        // 必须通过锁来保护条件谓语
        synchronized (lock){
            // 等待
            while(!conditionPredicate())
                lock.wait();
            // 合适状态操作
            ...
        }
    }
    ```

  其中，wait方法被while循环包裹是因为当wait返回时条件谓语并不为真时，可以让wait继续等待

  - **通知**

    每当在等待一个条件时，就需要确保在条件谓语变成真时通过某种方式进行通知

    在条件队列中可以通过notify和notifyAll，其中前者是通知条件队列中一个线程、后者是所有，但无论使用谁，都需要持有和条件队列对象相关联的锁

    下面是条件通知的标准形式

    ```java
    synchronized(lock){
    // 可能改变条件谓语状态的操作
    ...
    // 条件通知
    if(状态切换)
       lock.notifyAll(); 
        }
        
    ```

    一般情况下尽量使用notifyAll而不是notify，防止信号丢失，除非条件队列上的所有等待线程类型相同且符合单进单出（每次通知，最多唤醒一个线程执行）

## 2.ReentrantLock