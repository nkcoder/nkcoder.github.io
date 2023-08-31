---
title: "Java并发：可重入锁ReentrantLock"
date: 2020-10-23T08:16:59+08:00
categories:
- Java并发
draft: false
---
### 可重入性

可重入锁ReentrantLock，表示一个线程可以反复获得同一把锁。需要注意的是，如果一个线程多次获得锁，那么在释放的时候，也必须释放相同的次数。如果释放的次数少了，相当于当前这个线程还持有锁，其它线程无法获取锁，如果释放的次数多了，会抛出`java.lang.IllegalMonitorStateException`异常。

```java
LOCK.lock();
LOCK.lock();
try {
  i++;
} finally {
  LOCK.unlock();
  LOCK.unlock();
}
```

---

### 可中断

通过`lock()`方法获取锁的时候，如果锁已被其它线程占用，当前线程会休眠（不能被线程调度器调度），直到获取到锁。

`lockInterruptibly`方法允许线程在等待锁的过程中被其它线程中断。

```java
try {
	lock.lockInterruptibly();
} catch (InterruptedException exception) {
	// log exception
} finally {
	if (lock.isHeldByCurrentThread()) {
  	lock.unlock();
}
```

---

### 限时等待

`tryLock()`表示线程尝试获取锁，如果获取成功，则占用，并立即返回true；如果获取失败，也不会等待，而是立即返回false。支持可以带时间参数，表示限时等待，如`tryLock(3, TimeUnit.SECONDS)`等待3秒，超时则失败返回false。

```java
try {
  if (lock.tryLock(3, TimeUnit.SECONDS)) {
    TimeUnit.SECONDS.sleep(5);
    System.out.println(Thread.currentThread().getId() + ": my job done.");
  } else {
    System.out.println(Thread.currentThread().getId() + ": get lock failed.");
  }
} catch (InterruptedException exception) {
  exception.printStackTrace();
} finally {
  if (lock.isHeldByCurrentThread()) {
    lock.unlock();
  }
}
```

>注意：`tryLock()`将总是立即获得锁，只要锁是可用的（即锁没有被其它线程占用），即使该重入锁被设置为公平锁。

---

### 公平锁

公平锁，根据申请锁的先后顺序，先到先得。非公平锁，则是从所有等待锁的线程中，随机选择。

公平锁最大的特点是：不会导致线程饥饿，即只要排队，线程总是会申请到锁资源的。但是公平锁要求维护一个优先级队列，因此实现成本较高，而且性能较低下。

默认情况下，锁是非公平的，没有特殊需求，**不要**使用公平锁。使用`synchronized`关键字产生的锁就是非公平锁。重入锁允许对公平性进行设置，构造函数的参数如果是true，则表示公平锁。

```java
private ReentrantLock fairLock = new ReentrantLock(true);
```

### Condition

`Condition`的`await()`和`signal()/signalAll()`与Object的`wait()`和`notify()/notifyAll()`的用法和作用非常相似，只不过`wait()`和`notify()/notifyAll()`是与`synchronized`关键字配合使用的，而`Condition`的`await()`和`signal()/signalAll()`则是与`ReentrantLock`一起使用的。

当线程使用`Condition.await()`时，要求线程必须持有相关的重入锁，在`Condition.await()`被调用后，线程会释放锁。同理，在线程调用`Condition.signal()`时，也要求线程持有锁，`Condition.signal()`被调用后，系统会从当前`Condition`对象的等待队列中唤醒一个线程，线程被唤醒后，会尝试重新获取与之绑定的重入锁，一旦获取成功，则继续执行。所以线程在调用`Condition.signal()`要释放锁，否则被唤醒的线程无法重新获取锁。
