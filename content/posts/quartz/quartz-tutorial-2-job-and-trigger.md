---
title: Quartz系列二：API, Job和Trigger
description: Quartz系列文章的第二篇：介绍基础的API、Job类和Trigger类
date: 2020-02-19T21:07:06+08:00
categories:
- quartz
draft: false
---

## Quartz API

Quartz API核心接口有：

- Scheduler – 与scheduler交互的主要API；
- Job – 你通过scheduler执行任务，你的任务类需要实现的接口；
- JobDetail – 定义Job的实例；
- Trigger – 触发Job的执行；
- JobBuilder – 定义和创建JobDetail实例的接口;
- TriggerBuilder – 定义和创建Trigger实例的接口；

Scheduler的生命期，从SchedulerFactory创建它时开始，到Scheduler调用shutdown()方法时结束；Scheduler被创建后，可以增加、删除和列举Job和Trigger，以及执行其它与调度相关的操作（如暂停Trigger）。但是，Scheduler只有在调用start()方法后，才会真正地触发trigger（即执行job），见教程一。

Quartz提供的“builder”类，可以认为是一种领域特定语言（DSL，Domain Specific Language)。教程一中有相关示例，这里是其中的代码片段：（校对注：这种级联的API非常方便用户使用，大家以后写对外接口时也可以使用这种方式）

```java
// define the job and tie it to our HelloJob class
JobDetail job = newJob(HelloJob.class)
    .withIdentity("myJob", "group1") // name "myJob", group "group1"
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

定义job的代码使用的是从JobBuilder静态导入的方法。同样，定义trigger的代码使用的是从TriggerBuilder静态导入的方法 。 另外，也导入了SimpleSchedulerBuilder类的静态方法；

从DSL里静态导入的语句如下：

```java
import static org.quartz.JobBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.CalendarIntervalScheduleBuilder.*;
import static org.quartz.TriggerBuilder.*;
import static org.quartz.DateBuilder.*;
```

SchedulerBuilder接口的各种实现类，可以定义不同类型的调度计划（schedule）；

DateBuilder类包含很多方法，可以很方便地构造表示不同时间点的java.util.Date实例（如定义下一个小时为偶数的时间点，如果当前时间为9:43:27，则定义的时间为10:00:00）。

## Job和Trigger

一个job就是一个实现了Job接口的类，该接口只有一个方法：

Job接口：

```java
package org.quartz;

public interface Job {

public void execute(JobExecutionContext context)
    throws JobExecutionException;
}
```

当job的一个trigger被触发后（稍后会讲到），execute()方法会被scheduler的一个工作线程调用；传递给execute()方法的JobExecutionContext对象中保存着该job运行时的一些信息 ，执行job的scheduler的引用，触发job的trigger的引用，JobDetail对象引用，以及一些其它信息。

JobDetail对象是在将job加入scheduler时，由客户端程序（你的程序）创建的。它包含job的各种属性设置，以及用于存储job实例状态信息的JobDataMap。本节是对job实例的简单介绍，更多的细节将在下一节讲到。

Trigger用于触发Job的执行。当你准备调度一个job时，你创建一个Trigger的实例，然后设置调度相关的属性。Trigger也有一个相关联的JobDataMap，用于给Job传递一些触发相关的参数。Quartz自带了各种不同类型的Trigger，最常用的主要是SimpleTrigger和CronTrigger。

SimpleTrigger主要用于一次性执行的Job（只在某个特定的时间点执行一次），或者Job在特定的时间点执行，重复执行N次，每次执行间隔T个时间单位。CronTrigger在基于日历的调度上非常有用，如“每个星期五的正午”，或者“每月的第十天的上午10:15”等。

为什么既有Job，又有Trigger呢？很多任务调度器并不区分Job和Trigger。有些调度器只是简单地通过一个执行时间和一些job标识符来定义一个Job；其它的一些调度器将Quartz的Job和Trigger对象合二为一。在开发Quartz的时候，我们认为将调度和要调度的任务分离是合理的。在我们看来，这可以带来很多好处。

例如，Job被创建后，可以保存在Scheduler中，与Trigger是独立的，同一个Job可以有多个Trigger；这种松耦合的另一个好处是，当与Scheduler中的Job关联的trigger都过期时，可以配置Job稍后被重新调度，而不用重新定义Job；还有，可以修改或者替换Trigger，而不用重新定义与之关联的Job。

## Identities

将Job和Trigger注册到Scheduler时，可以为它们设置key，配置其身份属性。Job和Trigger的key（JobKey和TriggerKey）可以用于将Job和Trigger放到不同的分组（group）里，然后基于分组进行操作。同一个分组下的Job或Trigger的名称必须唯一，即一个Job或Trigger的key由名称（name）和分组（group）组成。

对于Job和Trigger，你现在有一个大概的了解了，更详细的介绍参见教程三(Jobs和JobDetails)和教程四(Triggers)。

- 原文链接：[Lesson 2: The Quartz API, Jobs And Triggers](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-02.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)