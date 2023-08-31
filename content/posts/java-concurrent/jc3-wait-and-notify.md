---
title: "Java并发：wait和notify"
date: 2020-10-22T23:42:44+08:00
categories:
- Java并发
draft: false
---
`wait()`和`notify()`都是`Object`对象上的方法，因此可以在任何对象上调用。

在一个对象上调用`wait()`方法后，线程就进入对象的等待队列，即线程在该对象上等待，直到其它线程调用了该对象的`notify()`或`notifyAll()`方法。

一个对象的等待队列可能有多个线程，即多个线程都在等待这个对象。`notify()`会随机唤醒等待队列上的一个线程，而`notifyAll()`则会唤醒等待队列上的所有线程。

必须在获得对象锁的前提下（即在`synchronized`语句中）才可以调用对象的`wait()`和`notify()`方法。

`wait()`方法在执行前需先获得对象锁，执行后会自动释放对象锁；`notify()`方法在执行前需先获得对象锁，但并不会直接释放锁，而是正常执行。被唤醒的线程并不能立即继续执行，在执行前还是需要先获得对象锁。

```java
import java.util.concurrent.TimeUnit;

public class WaitNotifyDemo {
  public static void main(String[] args) {
    final Object object = new Object();

    Thread t1 = new Thread("t1") {
      @Override
      public void run() {
        synchronized (object) {
          System.out.println(
              Thread.currentThread().getName() + ", start at " + System.currentTimeMillis());
          try {
            object.wait();
          } catch (InterruptedException e) {
            System.out.println(e.getStackTrace());
          }
          System.out
              .println(Thread.currentThread().getName() + ", end at " + System.currentTimeMillis());
        }
      }
    };

    Thread t2 = new Thread("t2") {
      @Override
      public void run() {
        synchronized (object) {
          System.out.println(
              Thread.currentThread().getName() + ", start at " + System.currentTimeMillis());
          object.notify();
          try {
            TimeUnit.SECONDS.sleep(2);
          } catch (InterruptedException ex) {
            System.out.println(ex.getStackTrace());
          }
          System.out
              .println(Thread.currentThread().getName() + ", end " + System.currentTimeMillis());
        }
      }
    };
    
    t1.start();
    t2.start();
  }
}
```

