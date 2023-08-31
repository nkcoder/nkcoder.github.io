---
title: "Java并发：volatile与synchronized"
date: 2020-10-23T08:15:01+08:00
categories:
- Java并发
draft: false
---
锁提供了两种主要特性：互斥和可见性。

- 互斥：同一时刻只能一个线程持有某个特定的锁，并访问共享数据
- 可见性：一个线程在释放锁之前对共享数据的修改，对随后获得该锁的线程是可见的，

`synchronized`是一种锁实现，因此具有互斥和可见性，可以保证线程安全。而`volatile`变量只能保证可见性，不能代替锁用于线程安全。

`synchronized`通过加锁保证任何时刻只会有一个线程访问同步代码块。`synchronized`加锁主要有三种形式：

- 在锁定对象上加锁：只有获得对象的锁才可以进入代码块

  ```java
  synchronized (object) {
    ...
  }
  ```

- 在实例方法上加锁：只有获得当前实例对象的锁才可以进入代码块

  ```java
  public synchronized void increase() {
    base++;
  }
  ```

- 在静态方法上加锁：只有获得类的锁才可以进入代码块

  ```java
  public static synchronized void increase() {
    base++;
  }
  ```

`synchronized`除了线程同步、保证线程安全外，还可以保证线程间的可见性和有序性。

```java
public class VolatileSynchronizedDemo {
  public static void main(String[] args) throws InterruptedException {

    CalculateRunnable calculateThread = new CalculateRunnable();

    // they should reference the same runnable
    Thread t1 = new Thread(calculateThread);
    Thread t2 = new Thread(calculateThread);

    t1.start();
    t2.start();
    t1.join();
    t2.join();
  }

  public static class CalculateRunnable implements Runnable {
    private int base = 0;

    public synchronized void increase() {
      base++;
    }

    @Override
    public void run() {

      for (int i = 0; i < 100000; i++) {
        increase();
      }

      System.out.println("base: " + base + ", thread: " + Thread.currentThread().getName());
    }
  }
}

```

`volatile`变量提供线程安全需要同时满足以下两个条件：

- 对变量的写操作不依赖于变量的当前值
- 该变量没有包含在其它变量的不变式中

第一个限制条件使`volatile`变量不能用于线程安全的计数器，即`counter++`。因为这个操作实际上是：`读取-修改-写入`组成的组合操作，必须保证原子性，而`volatile`无法提供原子性。

第二个限制条件使`volatile`变量不能用于多个变量的比较中，如：`start <= end`等：

```java
@NotThreadSafe 
public class NumberRange {
    private volatile int lower, upper;
 
    public int getLower() { return lower; }
    public int getUpper() { return upper; }
 
    public void setLower(int value) { 
        if (value > upper) 
            throw new IllegalArgumentException(...);
        lower = value;
    }
 
    public void setUpper(int value) { 
        if (value < lower) 
            throw new IllegalArgumentException(...);
        upper = value;
    }
}
```

上述代码使用`volatile`变量`lower`和`upper`并不能保证线程安全，因为如果两个线程同时使用不同的值调用方法`setLower()`和`setUpper()`，则会导致数据不一致，如初始状态为`(0, 10)`，如果两个线程同时调用`setLower(8)`和`setUpper(5)`，则会破坏状态约束。

volatile的适用模式：

- 状态标志：用一个布尔型的volatile变量表示一个状态标志，用于指示发生了一个重要的一次性事件，比如初始化完成：

  ```java
    private volatile boolean initDone = false;
  
    public void initWorkDone() {
      this.initDone = true;
      System.out.println("init done.");
    }
  
    public void startToWork() {
      while (true) {
        if (initDone) {
          System.out.println("init done, start to work");
          break;
        }
      }
    }
  ```

- 独立观察：定期发布观察结果，如用一个volatile变量记录系统的最后登录用户/最后访问用户等：

  ```java
  public class UserManager {
      public volatile String lastUser;
   
      public boolean authenticate(String user, String password) {
          boolean valid = passwordIsValid(user, password);
          if (valid) {
              User u = new User();
              activeUsers.add(u);
              lastUser = user;
          }
          return valid;
      }
  }
  ```

参考：

- [正确使用 Volatile 变量](https://www.ibm.com/developerworks/cn/java/j-jtp06197.html)
