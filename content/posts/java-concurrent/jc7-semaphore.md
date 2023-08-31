---
title: "Java并发：Semaphore"
date: 2020-10-23T08:18:02+08:00
categories:
- Java并发
draft: false
---
`synchronized`和`ReentrantLock`，在任一时刻都只允许一个线程访问临界资源，而`Semaphore`可以允许多个线程同时访问临界资源。

构造信号量时需要指定许可数，即允许同时访问的线程数量。

```java
private static final Semaphore SEMAPHORE = new Semaphore(5);
```

> 说明：构造函数的第二个参数表示是否公平，即根据先进先出的顺序获取许可。

信号量的主要方法有：

```java
public void acquire();	// 获取许可，如果无法获取，则线程等待直到被中断或成功获取到许可
public void acquireUninterruptibly(); // 获取许可，如果无法获取，则线程等待直到成功获取到许可，不响应中断
public boolean tryAcquire();	// 尝试获取许可，成功则返回true，否则返回false
public boolean tryAcquire(long timeout, TimeUnit unit); // 尝试获取许可，如果无法获取，等待指定时间
public void release();	// 释放一个许可
```

> 注意：`tryAcquire()`与`ReentrantLock`的`tryLock()`方法类似，无论锁是否公平，线程总是会立即获取可用的许可，而不论是否有其它线程在等待。

```java
try {
  SEMAPHORE.acquire();
  TimeUnit.SECONDS.sleep(3);
  System.out.println(Thread.currentThread().getId() + ": I'm done.");
  // System.out.println("-------------------------------------------");
} catch (InterruptedException exception) {
  exception.printStackTrace();
} finally {
  SEMAPHORE.release();
}
```
