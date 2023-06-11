# 三种ThreadLocal详解
ThreadLocal在实际开发中经常被用于存储当前线程上下文信息，如登录用户信息、全局traceId以及一些需要线程间隔离的数据

这篇文章主要是针对ThreadLocal的问题，进行简要分析，以及延伸出来的另外两种跨线程/线程池ThreadLocal简单介绍

## ThreadLocal
在Thread对象中有字段 threadLocals（ThreadLocalMap） 用于存储 key(ThreadLocal对象，弱引用) value(ThreadLocal value)
这个map通过线性探测法解决hash冲突问题，每次get/set时可能还伴随着清理掉key为null的entry键值对
主要问题：

1. 当前线程的ThreadLocal无法透传到子线程或线程池中
2. 当使用ThreadLocal对象时没有显式remove时，可能导致内存泄露


## InheritableThreadLocal

InheritableThreadLocal 子线程在创建时会拷贝父线程中的inheritableThreadLocalMap
Entry会重新创建，并插入子线程inheritableThreadLocalMap中，value只是浅拷贝

存在的主要问题：
1. value为浅拷贝，如果value为对象，在子线程中对value的属性的修改，父线程中也会对应改变
2. 拷贝过程只会发生在线程创建初始化时，当子线程如果是在线程池中，如果线程复用不会销毁，所以只有在首次创建线程
时才会透传，当线程已全部创建完毕时，InheritableThreadLocal是无法进行透传到线程池中的线程的
代码输出

## TransmittableThreadLocal
如果需要将ThreadLocal对象透传到线程池Runnable中使用，需要使用TransmittableThreadLocal

且需要配合TtlRunnable/TtlCallable使用，在原本Runnable中wrap一层

## 代码实验示例

```java
package com.passer.threadlocal;

import com.alibaba.ttl.TransmittableThreadLocal;
import com.alibaba.ttl.TtlRunnable;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author passer
 * @time 2023/6/11 12:13
 */
public class ThreadLocalDemo {
    public static void main(String[] args) throws Exception {
        testInheritableThreadLocal();
//        testTransmittableThreadLocal();
    }

    /**
     * <pre>
     * 在Thread对象中有字段 threadLocals（ThreadLocalMap） 用于存储 key(ThreadLocal对象，弱引用) value(ThreadLocal value)
     * 这个map通过线性探测法解决hash冲突问题，每次get/set时可能还伴随着清理掉key为null的entry键值对
     * 主要问题：
     * 1. 当前线程的ThreadLocal无法透传到子线程或线程池中
     * 2. 当使用ThreadLocal对象时没有显式remove时，可能导致内存泄露
     * </pre>
     */
    public static void testThreadLocal() {
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        threadLocal.set("hello");

        System.out.println(threadLocal.get());
    }

    /**
     * <pre>
     * 如果需要将ThreadLocal对象透传到线程池Runnable中使用，需要使用TransmittableThreadLocal
     * 且需要配合TtlRunnable/TtlCallable使用，在原本Runnable中wrap一层
     *
     * 代码输出
     * =================================
     * class com.alibaba.ttl.TransmittableThreadLocal TEST
     * pool-1-thread-1 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * set local pool-1-thread-1
     * pool-1-thread-2 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * set local pool-1-thread-2
     * pool-1-thread-3 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * set local pool-1-thread-3
     * ==================
     * main --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * ==================
     * pool-1-thread-1 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * pool-1-thread-1 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * pool-1-thread-2 --> class com.alibaba.ttl.TransmittableThreadLocal TEST, threadName: main
     * </pre>
     */
    public static void testTransmittableThreadLocal() throws Exception {
        TransmittableThreadLocal<String> transmittableThreadLocal = new TransmittableThreadLocal<>();
        testThreadLocal(transmittableThreadLocal);
    }


    /**
     * <pre>    
     * InheritableThreadLocal 子线程在创建时会拷贝父线程中的inheritableThreadLocalMap
     * Entry会重新创建，并插入子线程inheritableThreadLocalMap中，value只是浅拷贝
     * 存在的主要问题：
     * 1. value为浅拷贝，如果value为对象，在子线程中对value的属性的修改，父线程中也会对应改变
     * 2. 拷贝过程只会发生在线程创建初始化时，当子线程如果是在线程池中，如果线程复用不会销毁，所以只有在首次创建线程
     * 时才会透传，当线程已全部创建完毕时，InheritableThreadLocal是无法进行透传到线程池中的线程的
     * 代码输出
     * =================================
     * class java.lang.InheritableThreadLocal TEST 
     * pool-1-thread-1 --> class java.lang.InheritableThreadLocal TEST, threadName: main
     * set local pool-1-thread-1
     * pool-1-thread-2 --> class java.lang.InheritableThreadLocal TEST, threadName: main
     * set local pool-1-thread-2
     * pool-1-thread-3 --> class java.lang.InheritableThreadLocal TEST, threadName: main
     * set local pool-1-thread-3
     * ==================
     * main --> class java.lang.InheritableThreadLocal TEST, threadName: main
     * ==================
     * pool-1-thread-1 --> set new TEST, threadName: pool-1-thread-1
     * pool-1-thread-1 --> set new TEST, threadName: pool-1-thread-1
     * pool-1-thread-2 --> set new TEST, threadName: pool-1-thread-2
     * </pre>
     */
    public static void testInheritableThreadLocal() throws Exception{
        InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
        testThreadLocal(inheritableThreadLocal);
    }

    public static void testThreadLocal(ThreadLocal threadLocal) throws Exception{
        System.out.println("=================================");
        System.out.println(threadLocal.getClass() + " TEST ");
        CountDownLatch countDownLatch1 = new CountDownLatch(3);
        threadLocal.set(threadLocal.getClass() + " TEST, threadName: " + Thread.currentThread().getName());
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for (int i = 0; i < 3; i++) {
            executorService.submit(() -> {
                System.out.println(Thread.currentThread().getName() + " --> " + threadLocal.get());
                threadLocal.set("set new TEST, threadName: " + Thread.currentThread().getName());
                System.out.println("set local " + Thread.currentThread().getName());
                countDownLatch1.countDown();
            });
        }
        countDownLatch1.await();

        System.out.println("==================");
        System.out.println(Thread.currentThread().getName() + " --> " + threadLocal.get());
        System.out.println("==================");

        CountDownLatch countDownLatch2 = new CountDownLatch(3);
        for (int i = 0; i < 3; i++) {
            executorService.submit(TtlRunnable.get(() -> {
                System.out.println(Thread.currentThread().getName() + " --> " + threadLocal.get());
                countDownLatch2.countDown();
            }));
        }
        countDownLatch2.await();
        executorService.shutdown();
    }
}

```

