---
title: "Java并发：CountDownLatch"
date: 2020-10-23T08:19:55+08:00
categories:
- Java并发
draft: false
---
`CountDownLatch`可以被称为倒计数器，是一个线程协作工具，可以让一个（或一组线程）等待倒计数器（持有倒计数器的线程执行某项工作）结束，然后开始执行。

```java
CountDownLatch countDownLatch = new CountDownLatch(10);
```

`CountDownLatch`的参数表示计数器的值，线程调用`countdown()`方法使计数器递减，调用`await(`方法进行等待，当计数器的值为0时，等待的线程继续执行。

注意：

- 线程调用`countdown()`后，并不会阻塞，而是会继续执行
- 线程调用`await()`后会一直阻塞直到计数器的值为0，或者线程被中断
- `CountDownLatch`不能复用

实现：

- `CountDownLatch`内部有一个AQS（AbstractQueuedSynchronizer）的实现Sync，通过AQS的state参数保存计数器
- `await()`函数，先检查线程是否已被中断，然后检查计数器（即state）是否为0，如果为0，则当前线程继续向后执行，如果不为0，则将当前线程加入到等待队列中
- `countdown()`函数，将计数器的值减1，如果减1后，计数器的值为0，则释放所有等待的线程，如果在减1之前计数器的值已经为0，则什么都不做。
