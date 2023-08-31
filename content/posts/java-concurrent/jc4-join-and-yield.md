---
title: "Java并发：join和yield"
date: 2020-10-23T08:12:17+08:00
categories:
- Java并发
draft: false
---
调用线程的`join()`方法，表示等待该线程执行完后，当前线程再继续执行。

从实现上看，`join()`就是调用当前线程的`wait()`方法进行有限或无限的等待，线程执行完后再调用`notifyAll()`方法将等待的线程唤醒。

```java
public final synchronized void join(long millis)
  throws InterruptedException {
  long base = System.currentTimeMillis();
  long now = 0;

  if (millis < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (millis == 0) {
    while (isAlive()) {
      wait(0);
    }
  } else {
    while (isAlive()) {
      long delay = millis - now;
      if (delay <= 0) {
        break;
      }
      wait(delay);
      now = System.currentTimeMillis() - base;
    }
  }
}
```

因为实现上调用的是当前线程的`wait()`和`notifyAll()`，所以应用程序不要直接使用线程的`wait()`和`notifyAll()`，以免影响API的使用。

```java
public class JoinYieldDemo {
  private volatile static int total = 0;

  public static void main(String[] args) throws InterruptedException {

    Thread t1 = new Thread() {
      @Override
      public void run() {
        for (int i = 0; i < 100; i++) {
          total += i;
        }
      }
    };

    t1.start();
    t1.join();

    System.out.println("total: " + total);
  }
}
```

`Thread`类还有个静态方法：`yield()`，表示线程愿意让出CPU。但是让出CPU，并不表示线程不再执行了，还是会进行资源的争夺，只是能否被再次分配资源就不一定了。

如果线程不太重要，或者优先级较低，而且担心它占用太多CPU资源，可以在适当的时候调用`yield()`方法，给其它更重要的线程更多的执行机会。

