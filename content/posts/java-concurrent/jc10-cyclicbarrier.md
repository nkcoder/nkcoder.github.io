---
title: "Java并发：CyclicBarrier"
date: 2020-10-23T08:20:39+08:00
categories:
- Java并发
draft: false
---
`CyclicBarrier`可以称为循环栅栏，也是一个线程同步工具，在所有线程到达barrier之前，线程需要等待。

`CyclicBarrier`的必需参数为parties，可以称为同伴（线程）的数量：

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(10);		
```

当线程调用`await()`时，表示线程已到达，如果所有线程都已到达，则所有线程继续向后执行，否则线程等待，直到：

- 最后一个线程到达
- 等待中的某个线程被中断
- 等待中的某个线程在等待的过程中超时
- 等待中的某个线程调用了`CyclicBarrier`的`reset()`方法

如果某个等待的线程被中断，则所有其它等待的线程都会抛出`BrokenBarrierException`。

构造`CyclicBarrier`时还可以带第二个参数，是一个`Runnable`，即当所有线程都到达barrier时，需要执行的操作。

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(10, action);
```

该操作由最后一个到达barrier的线程会执行，当它执行完之后，其它等待的线程才被允许继续执行。

`CyclicBarrier`被称为循环栅栏，是因为它是可以循环使用的。

**实现：**

- 使用两个变量parties和count保存barrier关联的线程数量，parties用于barrier重复利用时重置线程数，而count用于记录到达barrier的线程的数量，当线程调用`await()`时，count的值递减，当count的值为0时，表示barrier完成，
- 线程调用`await()`，需要加锁，使用`ReentrantLock`实现。count递减，并判断是否为0：
  - 如果此时count为0，首先如果Runnable不为空，则在当前线程中执行。然后开始新的barrier循环（先将所有等待的线程唤醒，然后使用parties重置count。）
  - 如果count不为0，当前线程等待。