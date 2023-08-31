---
title: Quartz系列一：使用Quartz
description: Quartz系列文章的第一篇：入门
date: 2020-02-19T21:04:43+08:00
categories:
- quartz
draft: false
---

Scheduler在使用之前需要实例化。一般通过SchedulerFactory来创建一个实例。有些用户将factory的实例保存在JNDI中，但直接初始化，然后使用该实例也许更简单（见下面的示例）。

scheduler实例化后，可以启动(start)、暂停(stand-by)、停止(shutdown)。注意：scheduler被停止后，除非重新实例化，否则不能重新启动；只有当scheduler启动后，即使处于暂停状态也不行，trigger才会被触发（job才会被执行）。

下面的代码片段，实例化并启动一个scheduler，调度执行一个job：

```java
SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();

Scheduler sched = schedFact.getScheduler();

sched.start();

// define the job and tie it to our HelloJob class
JobDetail job = newJob(HelloJob.class)
    .withIdentity("myJob", "group1")
    .build();

// Trigger the job to run now, and then every 40 seconds
Trigger trigger = newTrigger()
    .withIdentity("myTrigger", "group1")
    .startNow()
    .withSchedule(simpleSchedule()
        .withIntervalInSeconds(40)
        .repeatForever())
    .build();

// Tell quartz to schedule the job using our trigger
sched.scheduleJob(job, trigger);
```

你看到了，quartz的使用并不难。教程二会简要地介绍job和trigger，以及quartz的API，然后你会更好地理解该示例。

- 原文链接：[Lesson 1: Using Quartz](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-01.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)