---
title: 'Semaphore-CountDownLatch-CyclicBarrier'
date: 2019-04-14
permalink: /posts/2019/04/Semaphore-CountDownLatch-CyclicBarrier/
excerpt: '三种同步辅助工具类'
tags:
  - Semaphore
  - CountDownLatch
  - CyclicBarrier
  - 同步工具类
---

## CountDownLatch （倒计时器）
- CountDownLatch是一个同步工具类，它允许一个或多个线程等待其他线程完成操作。
- 用给定的计数初始化CountDownLatch。由于调用了countDown()方法，所以在当前计数到达零之前，await()方法会一直受阻塞。之后，会释放所有等待的线程，await()的所有后续调用都将立即返回。这种现象只出现一次计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。
- CountDownLatch是一个通用同步工具，它有很多用途。将计数1初始化的CountDownLatch用作一个简单的开/关锁存器，或入口：在通过调用countDown()的线程打开入口前，所有调用 await()的线程都一直在入口处等待。用N初始化的CountDownLatch可以使一个线程在N个线程完成某项操作之前一直等待，或者使其在某项操作完成N次之前一直等待。
- CountDownLatch的一个有用特性是，它不要求调用countDown()方法的线程等到计数到达零时才继续，而在所有线程都能通过之前，它只是阻止任何线程继续通过一个await()。

### 常用方法
```c
CountDownLatch(int count) //构造一个用给定计数初始化的 CountDownLatch。
void	await() //使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。
boolean	await(long timeout, TimeUnit unit) //使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。
void	countDown() //递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
long	getCount() //返回当前计数。
```

### CountDownLatch 的三种典型用法

①某一线程在开始运行前等待n个线程执行完毕。将 CountDownLatch 的计数器初始化为n ：`new CountDownLatch(n) `，每当一个任务线程执行完毕，就将计数器减1 `countdownlatch.countDown()`，当计数器的值变为0时，在`CountDownLatch上 await()` 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。
②实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 `CountDownLatch` 对象，将其计数器初始化为 1 ：`new CountDownLatch(1) `，多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用 countDown() 时，计数器变为0，多个线程同时被唤醒。
③死锁检测：一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

### CountDownLatch 的使用示例

```java
public class CountDownLatchExample1 {
  // 请求的数量
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Lambda 表达式的运用
        try {
          test(threadnum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } finally {
          countDownLatch.countDown();// 表示一个请求已经被完成
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// 模拟请求的耗时操作
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// 模拟请求的耗时操作
  }
}

```
上面的代码中，我们定义了请求的数量为550，当这550个请求被处理完成之后，才会执行`System.out.println("finish");`。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 CountDownLatch.countDown()方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

### 内存一致性效果
线程中调用 countDown() 之前的操作 happen-before 紧跟在从另一个线程中对应 await() 成功返回的操作。

### CountDownLatch 的不足

CountDownLatch是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。

### CountDownLatch常见面试题：

解释一下CountDownLatch概念？
CountDownLatch 和CyclicBarrier的不同之处？
给出一些CountDownLatch使用的例子？
CountDownLatch 类中主要的方法？

## CyclicBarrier(循环栅栏)

CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。
CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。CyclicBarrier默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用`await`方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。

### 方法
- 构造方法
```java
CyclicBarrier(int parties) //创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
CyclicBarrier(int parties, Runnable barrierAction) //创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。  
```
- 普通方法
```c
 int	await() //在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。
 int	await(long timeout, TimeUnit unit) //在所有参与者都已经在此屏障上调用 await 方法之前将一直等待,或者超出了指定的等待时间。
 int	getNumberWaiting() //返回当前在屏障处等待的参与者数目。
 int	getParties() //返回要求启动此 barrier 的参与者数目。
 boolean	isBroken() //查询此屏障是否处于损坏状态。
 void	reset() //将屏障重置为其初始状态。
```

### CyclicBarrier 的应用场景

CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

### CyclicBarrier详解

一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

CyclicBarrier 支持一个可选的 Runnable 命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次。若在继续所有参与线程之前更新共享状态，此屏障操作 很有用。

示例用法：下面是一个在并行分解设计中使用 barrier 的例子：

```java
class Solver {
	final int N;
	final float[][] data;
	final CyclicBarrier barrier;
	class Worker implements Runnable {
		int myRow;

		Worker(int row) {
			myRow = row;
		}

		public void run() {
			while (!done()) {
				processRow(myRow);

				try {
					barrier.await();
				} catch (InterruptedException ex) {
					return;
				} catch (BrokenBarrierException ex) {
					return;
				}
			}
		}
	}
	public Solver(float[][] matrix) {
	     data = matrix;
	     N = matrix.length;
	     barrier = new CyclicBarrier(N,
	                                 new Runnable() {
	                                   public void run() {
	                                     mergeRows(...);
	                                   }
	                                 });
	     for (int i = 0; i < N; ++i)
	       new Thread(new Worker(i)).start();

	     waitUntilDone();
	   }
}
```
在这个例子中，每个 worker 线程处理矩阵的一行，在处理完所有的行之前，该线程将一直在屏障处等待。处理完所有的行之后，将执行所提供的 Runnable 屏障操作，并合并这些行。如果合并者确定已经找到了一个解决方案，那么 done() 将返回 true，所有的 worker 线程都将终止。
如果屏障操作在执行时不依赖于正挂起的线程，则线程组中的任何线程在获得释放时都能执行该操作。为方便此操作，每次调用 await() 都将返回能到达屏障处的线程的索引。然后，您可以选择哪个线程应该执行屏障操作，例如：

```java
if (barrier.await() == 0) {
   // log the completion of this iteration
 }
```
对于失败的同步尝试，CyclicBarrier 使用了一种要么全部要么全不 (all-or-none) 的破坏模式：如果因为中断、失败或者超时等原因，导致线程过早地离开了屏障点，那么在该屏障点等待的其他所有线程也将通过 BrokenBarrierException（如果它们几乎同时被中断，则用 InterruptedException）以反常的方式离开。

### 内存一致性效果

线程中调用 await() 之前的操作 happen-before 那些是屏障操作的一部份的操作，后者依次 happen-before 紧跟在从另一个线程中对应 await() 成功返回的操作。

## CyclicBarrier和CountDownLatch的区别

CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置。所以CyclicBarrier能够处理更复杂的业务场景。
我们来从jdk作者设计的目的来看，javadoc是这么描述它们的：

> CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.(CountDownLatch: 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；)
> CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.(CyclicBarrier : 多个线程互相等待，直到到达同一个同步点，再继续一起执行。)

对于CountDownLatch来说，重点是“一个线程（多个线程）等待”，而其他的N个线程在完成“某件事情”之后，可以终止，也可以等待。而对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。
CountDownLatch是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而CyclicBarrier更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

![CyclicBarrier和CountDownLatch的区别](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/Java%20%E7%A8%8B%E5%BA%8F%E5%91%98%E5%BF%85%E5%A4%87%EF%BC%9A%E5%B9%B6%E5%8F%91%E7%9F%A5%E8%AF%86%E7%B3%BB%E7%BB%9F%E6%80%BB%E7%BB%93/AQS333.png)

CyclicBarrier和CountDownLatch的区别这部分内容参考了如下两篇文章：

- https://blog.csdn.net/u010185262/article/details/54692886
- https://blog.csdn.net/tolcf/article/details/50925145?utm_source=blogxgwz0

## Semaphore(信号量)-允许多个线程同时访问

**synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。**  
示例代码如下：

```java
public class SemaphoreExample1 {
  // 请求的数量
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    // 一次只能允许执行的线程数量。
    final Semaphore semaphore = new Semaphore(20);

    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Lambda 表达式的运用
        try {
          semaphore.acquire();// 获取一个许可，所以可运行线程数量为20/1=20
          test(threadnum);
          semaphore.release();// 释放一个许可
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }

      });
    }
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// 模拟请求的耗时操作
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// 模拟请求的耗时操作
  }
}
```

执行 `acquire` 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个 `release` 方法增加一个许可证，这可能会释放一个阻塞的acquire方法。然而，其实并没有实际的许可证这个对象，Semaphore只是维持了一个可获得许可证的数量。 Semaphore经常用于限制获取某种资源的线程数量。

当然一次也可以一次拿取和释放多个许可，不过一般没有必要这样做：

```java
          semaphore.acquire(5);// 获取5个许可，所以可运行线程数量为20/5=4
          test(threadnum);
          semaphore.release(5);// 获取5个许可，所以可运行线程数量为20/5=4
```

除了 `acquire`方法之外，另一个比较常用的与之对应的方法是`tryAcquire`方法，该方法如果获取不到许可就立即返回false。


Semaphore 有两种模式，公平模式和非公平模式。

- **公平模式：** 调用acquire的顺序就是获取许可证的顺序，遵循FIFO；
- **非公平模式：** 抢占式的。

**Semaphore 对应的两个构造方法如下：**

```java
   public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```
**这两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。**

由于篇幅问题，如果对 Semaphore 源码感兴趣的朋友可以看下面这篇文章：

- https://blog.csdn.net/qq_19431333/article/details/70212663
