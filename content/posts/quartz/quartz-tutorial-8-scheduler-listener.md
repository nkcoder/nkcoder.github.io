---
title: Quartz系列八：SchedulerListener
description: Quartz系列文章的第八篇：介绍SchedulerListener
date: 2020-02-19T21:11:38+08:00
categories:
- quartz
draft: false
---

`SchedulerListener`与TriggerListener、JobListener类似，但它仅接收来自Scheduler自身的消息，而不一定是某个具体的trigger或job的消息。

scheduler相关的消息包括：job/trigger的增加、job/trigger的删除、scheduler内部发生的严重错误以及scheduler关闭的消息等；

**org.quartz.SchedulerListener接口：**

```java
public interface SchedulerListener {

    public void jobScheduled(Trigger trigger);

    public void jobUnscheduled(String triggerName, String triggerGroup);

    public void triggerFinalized(Trigger trigger);

    public void triggersPaused(String triggerName, String triggerGroup);

    public void triggersResumed(String triggerName, String triggerGroup);

    public void jobsPaused(String jobName, String jobGroup);

    public void jobsResumed(String jobName, String jobGroup);

    public void schedulerError(String msg, SchedulerException cause);

    public void schedulerStarted();

    public void schedulerInStandbyMode();

    public void schedulerShutdown();

    public void schedulingDataCleared();
}
```

SchedulerListener也是注册到scheduler的ListenerManager上的，任何实现了org.quartz.SchedulerListener接口的对象都可以是SchedulerListener。

**添加一个SchedulerListener：**

```java
scheduler.getListenerManager().addSchedulerListener(mySchedListener);
```

**删除一个SchedulerListener：**

```java
scheduler.getListenerManager().removeSchedulerListener(mySchedListener);
```

- 参考链接：[Lesson 8: SchedulerListeners](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-08.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)