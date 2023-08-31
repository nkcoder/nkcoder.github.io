---
title: "Java并发：线程中断"
date: 2020-10-22T23:34:07+08:00
categories:
- Java并发
draft: false
---
如何让线程停止或退出呢？

大多数线程执行完任务后就自动退出了，但也有部分后台线程是常驻的，不停地提供服务（通过`while`循环等）。

`Thread`类中的`suspend()`方法和`resume()`方法早已废弃，禁止使用了，因为容易导致死锁：如果目标线程被`suspend()`时持有关键资源的锁，而其它所有线程在获得该锁之前不能访问该关键资源。当可以`resume()`目标线程的线程在`resume()`目标线程之前尝试获取该关键资源的锁，此时死锁发生了。

`Thread`类的`stop()`方法同样被废弃，因为该方法强制目标线程立即停止执行，容易导致数据或状态不一致。当目标线程被`stop()`时，线程持有的所有锁都被立即释放，如果之前被锁保护的对象处于不一致状态（如数据处理到一半），则该对象就暴露给了其它线程，导致不可预期的结果。

停止线程更好的方式是使用`中断interrupt()`。

线程中断并不是让目标线程立即停止退出，而是给目标线程发个通知：有线程希望你退出啦！至于目标线程收到通知后如何处理，由目标线程自己决定。

Thread类有三个方法与中断有关：

- `Thread#isInterrupted()`：实例方法，判断线程是否被中断，中断状态不变
- `Thread#interrupt()`：实例方法，中断线程，默认设置线程的中断状态，如果线程当前处于`wait()`, `sleep()`, `join()`方法中，则线程会抛出`InterruptedException`异常，同时中断状态被清除
- `Thread.interrupted()`：静态方法，判断当前线程是否被中断，同时中断状态被清除

```java
public class InterruptDemo {
  public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(new MyRunnable());
    t1.start();
    Thread.sleep(3000);
    t1.interrupt();
  }

  public static class MyRunnable implements Runnable {

    @Override
    public void run() {
      while (true) {
        // when current thread is interrupted, stop
        if (Thread.currentThread().isInterrupted()) {
          System.out.println("I'm interrupted, exit.");
          break;
        }

        try {
          Thread.sleep(2000);
        } catch (InterruptedException ex) {
          // when interrupt in sleep()|join()|wait(), the interrupt status is cleared
          System.out.println("I'm interrupted when sleeping.");
          // interrupt again to set interrupt status
          Thread.currentThread().interrupt();;
        }
      }
    }
  }
}
```