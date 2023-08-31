---
title: Quartz系列七：TriggerListeners与JobListeners
description: Quartz系列文章的第七篇：介绍TriggerListeners与JobListeners
date: 2020-02-19T21:11:02+08:00
categories:
- quartz
draft: false
---


listener是一个对象，用于监听scheduler中发生的事件，然后执行相应的操作；你可能已经猜到了，`TriggerListeners`接受与trigger相关的事件，`JobListeners`接受与jobs相关的事件。

trigger相关的事件包括：trigger的触发、trigger错过触发(mis-fire)以及trigger的完成(即trigger触发的job执行完成)。

**org.quartz.TriggerListener接口:**

```java
public interface TriggerListener {

  public String getName();

  public void triggerFired(Trigger trigger, JobExecutionContext context);

  public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

   public void triggerMisfired(Trigger trigger);

    public void triggerComplete(Trigger trigger, JobExecutionContext context,
          int triggerInstructionCode);
}
```

job相关的事件包括：job即将执行的通知以及job执行完毕的通知。

**org.quartz.JobListener接口：**

```java
public interface JobListener {

  public String getName();

  public void jobToBeExecuted(JobExecutionContext context);

    public void jobExecutionVetoed(JobExecutionContext context);

 public void jobWasExecuted(JobExecutionContext context,
            JobExecutionException jobException);

}
```

## 使用自定义的listener

创建一个listener，只需要实现org.quartz.TriggerListener接口或者org.quartz.JobListener 接口即可；然后在运行时将listener注册到scheduler上，并且需要给listener取个名称(因为listener需要通过其getName()方法广播它的名称)。

我们可以实现上面提到的接口，但更方便的方式是继承JobListenerSupport类或者TriggerListenerSupport类，只需重写需要的方法即可。

listener是注册到scheduler的ListenerManager上的，与listener一同注册的还有一个Matcher对象，该对象用于描述listener期望接收事件的job或trigger。

    listener是在运行的时候注册到scheduler上的，而且不会与job和trigger一样保存在JobStore中。因为listener一般是应用的一个集成点(integration point)，因此，应用每次运行的时候，listener都应该重新注册到scheduler上。

**给一个job添加JobListener：**

```java
scheduler.getListenerManager().addJobListener(myJobListener, KeyMatcher.jobKeyEquals(new JobKey("myJobName", "myJobGroup")));
```

可以对matcher和key下的类进行静态导入，这样使得matcher的定义更加清晰：

```java
import static org.quartz.JobKey.*;
import static org.quartz.impl.matchers.KeyMatcher.*;
import static org.quartz.impl.matchers.GroupMatcher.*;
import static org.quartz.impl.matchers.AndMatcher.*;
import static org.quartz.impl.matchers.OrMatcher.*;
import static org.quartz.impl.matchers.EverythingMatcher.*;
...etc.
```

静态导入后，上面的实例可以写成：

```java
scheduler.getListenerManager().addJobListener(myJobListener, jobKeyEquals(jobKey("myJobName", "myJobGroup")));
```

给一个group下的所有job添加一个JobListener：

```java
scheduler.getListenerManager().addJobListener(myJobListener, jobGroupEquals("myJobGroup"));
```

给两个group下的所有job添加一个JobListener：

```java
scheduler.getListenerManager().addJobListener(myJobListener, or(jobGroupEquals("myJobGroup"), jobGroupEquals("yourGroup")));
```

给所有的job添加一个JobListener：

```java
scheduler.getListenerManager().addJobListener(myJobListener, allJobs());
```

TriggerListener的注册过程与JobListener类似。

Quartz的大部分用户可能都用不到listener，但如果应用对发生的事件感兴趣，使用listener会更方便，不需要job显式去通知应用。

- 原文链接：[Lesson 7: TriggerListeners and JobListeners](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-07.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)


