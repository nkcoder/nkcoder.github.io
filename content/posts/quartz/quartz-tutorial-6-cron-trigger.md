---
title: Quartz系列六：CronTrigger
description: Quartz系列文章的第六篇：介绍CronTrigger类
date: 2020-02-19T21:10:24+08:00
categories:
- quartz
draft: false
---

如果你需要的是基于日历表示法的调度，而不是基于指定间隔的简单调度，那么CronTrigger比SimpleTrigger更合适。

使用CronTrigger，你可以配置这样的调度：“每周五的中午”，或者“每个工作日的上午9:30”，或者“在一月的每个周一、周三和周五的上午9点到10点之间每隔5分钟”。

与SimpleTrigger一样，CronTrigger需要设置`startTime`属性，表示调度生效的时间，以及(可选的)`endTime`属性，表示调度的结束时间。

## Cron表达式

Cron表达式用于配置CronTrigger的实例，它由7个字段组成，字段之间由空格分开，它们表示的含义如下：

    1. 秒 (Seconds)
    2. 分钟 (Minutes)
    3. 小时 (Hours)
    4. 日(一个月的一天) (Day-of-Month)
    5. 月份   (Month)
    6. 周(一周的一天) (Day-of-Week)
    7. 年份(可选的) (Year)

一个完整的Cron表达式的例子如字符串：”0 0 12 ? * WED” - 表示“每周三的中午12:00:00”；

每一个字段可以包含范围或者列举。比如，上例中的周字段(即”WED“)可以被替换为：”MON-FRI”, “MON,WED,FRI”, 或者”MON-WED,SAT”。

通配符()表示该字段上每一个可取的值。因此，上例中月字段上的表示“每个月”。周字段上的*表示“一周的每一天”。

所有的字段都有一些可取的值的集合。有些就很明显 - 比如，秒和分钟字段可以取0到59的整数，小时字段可以取0到23的整数，日字段(Day-of-Month)可以取1到31的整数，不过你需要注意指定的月份到底有多少天。月字段可以取0到11之间的整数，或者使用字符串表示：JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV 和 DEC。周字段(Days-of-Week)可以取的值为1到7(1表示Sunday)，或者使用字符串表示：SUN, MON, TUE, WED, THU, FRI and SAT。

字符‘/’表示增量。比如，分钟字段使用”0/15“，表示”在这个小时内，从0分钟开始，每隔15分钟“；如果分钟字段使用”3/20“，则表示”在这个小时内，从第3分钟开始，每隔20分钟“，这与在分钟字段使用“3,23,43”表示的含义是一样的。注意一个细节，“/35”的含义并不是“每隔35分钟”，而是“在这个小时内，从第0分钟开始，每隔35分钟”。

字符’?’可以用于日字段和周字段，表示“没有具体的值“。它主要用于在这两个字段的某一个中指定某个值，更好的理解请参考下面的示例（以及CrontTrigger的JavaDoc）。

字符’L’可以用于日字段和周字段，它是last的缩写，但是用于这两个字段时，含义并不相同。比如，“L”用于日子段，表示“这个月的最后一天” - 即一月的第31天，非闰年二月的第28天。如果‘L’单独用于周字段，它的含义就是“7”或者“SAT”。但是，如果‘L’用于周字段时，前面还有一个值，则表示“这个月的最后一个周××” - 比如，“6L”或“FRIL”表示“这个月的最后一个周五”。我们也可以指定与这个月最后一天的偏移量，比如“L-3”表示与这个月的最后一天相差3天。需要注意的是，如果使用选项‘L’，则不要使用列举和范围，否则结果将是不可预料的。

字符’W’表示离某一天最近的工作日（weekday）。例如，日字段上的值“15W”表示“离这个月的15号最近的工作日”。

字符’#’表示“这个月的第n个周××”。例如，在周字段上的“6#3”或“FRI#3”表示“这个月的3个周五”。

以下为一些表达式的例子及对应的含义 - 在org.quartz.CronExpression的JavaDoc有更多的描述。

## Cron表达式示例

CronTrigger示例1 - 每隔5分钟执行一次：

    0 0/5 * * * ?

CronTrigger示例2 - 每隔5分钟，且在该分钟的第10秒执行（如10:00:10 am, 10:05:10 am等）：

    10 0/5 * * * ?

CronTrigger示例3 - 每周三和周五的10:30, 11:30, 12:30和13:30执行：

    0 30 10-13 ? * WED,FRI

Crontrigger示例4 - 在每个月的第5天和第20天，上午8点到上午10点之间每隔半小时执行。注意：该trigger不会在上午10点执行，只在上午8:00, 8:30, 9:00和9:30执行：

    0 0/30 8-9 5,20 * ?

提示一下，有些调度需求太复杂了，无法使用一个trigger表达 - 比如：“在上午9点和上午10点之间，每隔5分钟执行一次，且在下午1点到下午10点之间每隔20分钟执行一次”。应对这样的需求，可以创建两个trigger，注册到同一个job上即可。

## 创建CronTrigger

CronTrigger实例可以通过TriggerBuilder（配置主要属性）和CronScheduleBuilder（配置CronTrigger专有的属性）配置。为了以DSL风格使用这些builder，需要静态导入：

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

构建一个trigger，在每天的上午8点到下午5点之间每隔2分钟执行一次，如：

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
    .forJob("myJob", "group1")
    .build();
```

构建一个trigger，每天上午10:42执行：

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(dailyAtHourAndMinute(10, 42))
    .forJob(myJobKey)
    .build();
```

或者：

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 * * ?"))
    .forJob(myJobKey)
    .build();
```

构建一个trigger，在每周三的上午10:42执行，并配置TimeZone：

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42)
        .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles")))
    .forJob(myJobKey)
    .build();
```

或者：

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 ? * WED")
        .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles")))
    .forJob(myJobKey)
    .build();
```

## CronTriggter错过触发策略

下面的这些策略用于告诉Quartz，如果CronTrigger错过触发时应该采取的处理方式。（关于错过触发的场景，在本教程的Trigger介绍部分以经介绍过）。这些策略值是定义在CronTrigger上的常量，包括：

    MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
    MISFIRE_INSTRUCTION_DO_NOTHING
    MISFIRE_INSTRUCTION_FIRE_NOW

所有的trigger都可以使用Trigger.MISFIRE_INSTRUCTION_SMART_POLICY策略，该策略也是所有trigger的默认策略。在CronTrigger中，smart policy策略其实就是MISFIRE_INSTRUCTION_FIRE_NOW. CronTrigger.updateAfterMisfire()的API文档中对该策略有更详细的解释。

在构造CronTrigger时，将错过触发策略作为调度的一部分进行配置（通过CronScheduleBuilder):

```java
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?")
        .withMisfireHandlingInstructionFireAndProceed())
    .forJob("myJob", "group1")
    .build();
```

- 原文链接：[Lesson 6: CronTrigger](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-06.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)