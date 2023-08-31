---
title: Quartz系列五：SimpleTrigger
description: Quartz系列文章的第五篇：介绍SimpleTrigger类
date: 2020-02-19T21:09:37+08:00
categories:
- quartz
draft: false
---

`SimpleTrigger`可以满足的调度需求是：在具体的时间点执行一次，或者在具体的时间点执行，并且以指定的间隔重复执行若干次。比如，你有一个trigger，你可以设置它在2015年1月13日的上午11:23:54准时触发，或者在这个时间点触发，并且每隔2秒触发一次，一共重复5次。

根据描述，你可能已经发现了，SimpleTrigger的属性包括：开始时间、结束时间、重复次数以及重复的间隔。这些属性的含义与你所期望的是一致的，只是关于结束时间有一些地方需要注意。

重复次数，可以是0、正整数，以及常量SimpleTrigger.REPEAT_INDEFINITELY。重复的间隔，必须是0，或者long型的正数，表示毫秒。注意，如果重复间隔为0，trigger将会以重复次数并发执行(或者以scheduler可以处理的近似并发数)。

如果你还不熟悉DateBuilder，了解后你会发现使用它可以非常方便地构造基于开始时间(或终止时间)的调度策略。

endTime属性的值会覆盖设置重复次数的属性值；比如，你可以创建一个trigger，在终止时间之前每隔10秒执行一次，你不需要去计算在开始时间和终止时间之间的重复次数，只需要设置终止时间并将重复次数设置为REPEAT_INDEFINITELY(当然，你也可以将重复次数设置为一个很大的值，并保证该值比trigger在终止时间之前实际触发的次数要大即可)。

SimpleTrigger实例通过TriggerBuilder设置主要的属性，通过SimpleScheduleBuilder设置与SimpleTrigger相关的属性。要使用这些builder的静态方法，需要静态导入：

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

下面的例子，是基于简单调度(simple schedule)创建的trigger。建议都看一下，因为每个例子都包含一个不同的实现点：

**指定时间开始触发，不重复：**

```java
SimpleTrigger trigger = (SimpleTrigger) newTrigger() 
    .withIdentity("trigger1", "group1")
    .startAt(myStartTime)                     // some Date 
    .forJob("job1", "group1")                 // identify job with name, group strings
    .build();
```

**指定时间触发，每隔10秒执行一次，重复10次：**

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startAt(myTimeToStartFiring)  // if a start time is not given (if this line were omitted), "now" is implied
    .withSchedule(simpleSchedule()
        .withIntervalInSeconds(10)
        .withRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
    .forJob(myJob) // identify job with handle to its JobDetail itself                   
    .build();
```

**5分钟以后开始触发，仅执行一次：**

```java
trigger = (SimpleTrigger) newTrigger() 
    .withIdentity("trigger5", "group1")
    .startAt(futureDate(5, IntervalUnit.MINUTE)) // use DateBuilder to create a date in the future
    .forJob(myJobKey) // identify job with its JobKey
    .build();
```

**立即触发，每个5分钟执行一次，直到22:00：**

```java
trigger = newTrigger()
    .withIdentity("trigger7", "group1")
    .withSchedule(simpleSchedule()
        .withIntervalInMinutes(5)
        .repeatForever())
    .endAt(dateOf(22, 0, 0))
    .build();
```

**在下一小时整点触发，每个2小时执行一次，一直重复：**

```java
trigger = newTrigger()
    .withIdentity("trigger8") // because group is not specified, "trigger8" will be in the default group
    .startAt(evenHourDate(null)) // get the next even-hour (minutes and seconds zero ("00:00"))
    .withSchedule(simpleSchedule()
        .withIntervalInHours(2)
        .repeatForever())
    // note that in this example, 'forJob(..)' is not called which is valid 
    // if the trigger is passed to the scheduler along with the job  
    .build();

scheduler.scheduleJob(trigger, job);
```

请查阅TriggerBuilder和SimpleScheduleBuilder提供的方法，以便对上述示例中未提到的选项有所了解。

    TriggerBuilder(以及Quartz的其它builder)会为那些没有被显式设置的属性选择合理的默认值。比如：如果你没有调用withIdentity(..)方法，TriggerBuilder会为trigger生成一个随机的名称；如果没有调用startAt(..)方法，则默认使用当前时间，即trigger立即生效。

## SimpleTrigger Misfire策略

SimpleTrigger有几个misfire相关的策略，告诉quartz当misfire发生的时候应该如何处理。(Misfire策略参考教程四：Trigger介绍)。这些策略以常量的形式在SimpleTrigger中定义(JavaDoc中介绍了它们的功能)。这些策略包括：

**SimpleTrigger的Misfire策略常量：**

    MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
    MISFIRE_INSTRUCTION_FIRE_NOW
    MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT
    MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT
    MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT
    MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT

回顾一下，所有的trigger都有一个Trigger.MISFIRE_INSTRUCTION_SMART_POLICY策略可以使用，该策略也是所有trigger的默认策略。

如果使用smart policy，SimpleTrigger会根据实例的配置及状态，在所有MISFIRE策略中动态选择一种Misfire策略。SimpleTrigger.updateAfterMisfire()的JavaDoc中解释了该动态行为的具体细节。

在使用SimpleTrigger构造trigger时，misfire策略作为基本调度(simple schedule)的一部分进行配置(通过SimpleSchedulerBuilder设置)：

```java
trigger = newTrigger()
    .withIdentity("trigger7", "group1")
    .withSchedule(simpleSchedule()
        .withIntervalInMinutes(5)
        .repeatForever()
        .withMisfireHandlingInstructionNextWithExistingCount())
    .build();
```

- 原文链接：[Lesson 5: SimpleTrigger](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-05.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)
