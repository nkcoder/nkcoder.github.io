---
title: Quartz系列四：Trigger
description: Quartz系列文章的第四篇：进一步介绍Trigger类
date: 2020-02-19T21:08:49+08:00
categories:
- quartz
draft: false
---

与job一样，trigger也很容易使用，但是还有一些扩展选项需要理解，以便更好地使用quartz。trigger也有很多类型，可以根据实际需要来选择。

最常用的两种trigger会分别在教程五：SimpleTrigger和教程六：CronTrigger中讲到；

## Trigger的公共属性

所有类型的trigger都有TriggerKey这个属性，表示trigger的身份；除此之外，trigger还有很多其它的公共属性。这些属性，在构建trigger的时候可以通过TriggerBuilder设置。

trigger的公共属性有：

- jobKey属性：当trigger触发时被执行的job的身份；
- startTime属性：设置trigger第一次触发的时间；该属性的值是java.util.Date类型，表示某个指定的时间点；有些类型的trigger，会在设置的startTime时立即触发，有些类型的trigger，表示其触发是在startTime之后开始生效。比如，现在是1月份，你设置了一个trigger–“在每个月的第5天执行”，然后你将startTime属性设置为4月1号，则该trigger第一次触发会是在几个月以后了(即4月5号)。
- endTime属性：表示trigger失效的时间点。比如，”每月第5天执行”的trigger，如果其endTime是7月1号，则其最后一次执行时间是6月5号。
- 其它的属性，会在下面的小节中解释。

## 优先级(priority)

如果你的trigger很多(或者Quartz线程池的工作线程太少)，Quartz可能没有足够的资源同时触发所有的trigger；这种情况下，你可能希望控制哪些trigger优先使用Quartz的工作线程，要达到该目的，可以在trigger上设置priority属性。比如，你有N个trigger需要同时触发，但只有Z个工作线程，优先级最高的Z个trigger会被首先触发。如果没有为trigger设置优先级，trigger使用默认优先级，值为5；priority属性的值可以是任意整数，正数、负数都可以。

**注意**：只有同时触发的trigger之间才会比较优先级。10:59触发的trigger总是在11:00触发的trigger之前执行。

**注意**：如果trigger是可恢复的，在恢复后再调度时，优先级与原trigger是一样的。

## 错过触发(misfire)

trigger还有一个重要的属性misfire；如果scheduler关闭了，或者Quartz线程池中没有可用的线程来执行job，此时持久性的trigger就会错过(miss)其触发时间，即错过触发(misfire)。不同类型的trigger，有不同的misfire机制。它们默认都使用“智能机制(smart policy)”，即根据trigger的类型和配置动态调整行为。当scheduler启动的时候，查询所有错过触发(misfire)的持久性trigger。然后根据它们各自的misfire机制更新trigger的信息。当你在项目中使用Quartz时，你应该对各种类型的trigger的misfire机制都比较熟悉，这些misfire机制在JavaDoc中有说明。关于misfire机制的细节，会在讲到具体的trigger时作介绍。

## 日历(calendar)

Quartz的Calendar对象(不是java.util.Calendar对象)可以在定义和存储trigger的时候与trigger进行关联。Calendar用于从trigger的调度计划中排除时间段。比如，可以创建一个trigger，每个工作日的上午9:30执行，然后增加一个Calendar，排除掉所有的商业节日。

任何实现了Calendar接口的可序列化对象都可以作为Calendar对象，Calendar接口如下：

```java
package org.quartz;

public interface Calendar {

    public boolean isTimeIncluded(long timeStamp);

    public long getNextIncludedTime(long timeStamp);

}
```

注意到这些方法的参数类型为long。你也许猜到了，他们就是毫秒单位的时间戳。即Calendar排除时间段的单位可以精确到毫秒。你也许对“排除一整天”的Calendar比较感兴趣。Quartz提供的org.quartz.impl.HolidayCalendar类可以很方便地实现。

Calendar必须先实例化，然后通过addCalendar()方法注册到scheduler。如果使用HolidayCalendar，实例化后，需要调用addExcludedDate(Date date)方法从调度计划中排除时间段。以下示例是将同一个Calendar实例用于多个trigger：

```java
HolidayCalendar cal = new HolidayCalendar();
cal.addExcludedDate( someDate );
cal.addExcludedDate( someOtherDate );

sched.addCalendar("myHolidays", cal, false);

Trigger t = newTrigger()
    .withIdentity("myTrigger")
    .forJob("myJob")
    .withSchedule(dailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
    .modifiedByCalendar("myHolidays") // but not on holidays
    .build();

// .. schedule job with trigger

Trigger t2 = newTrigger()
    .withIdentity("myTrigger2")
    .forJob("myJob2")
    .withSchedule(dailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
    .modifiedByCalendar("myHolidays") // but not on holidays
    .build();

// .. schedule job with trigger2
```

构造trigger的细节将在接下来的几节中讲到。就现在，只需要知道上面的代码创建了两个trigger，都是每天执行一次。但是在Calendar定义的时间段内，trigger的执行被跳过。

org.quartz.impl.calendar包下有各种Calendar的实现，根据需要进行选择。

- 原文链接：[Lesson 4: More About Triggers](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-04.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)