---
title: Quartz系列三：使用Job和JobDetail
description: Quartz系列文章的第三篇：介绍Job和JobDetail
date: 2020-02-19T21:08:06+08:00
categories:
- quartz
draft: false
---

正如在教程二中讲到的，Job实现起来很容易，该接口只有一个“execute”方法。本节主要关注：Job的特点、Job接口的execute方法以及JobDetail。

你定义了一个实现Job接口的类，这个类仅仅表明该job需要完成什么类型的任务，除此之外，Quartz还需要知道该Job实例所包含的属性；这将由JobDetail类来完成。

JobDetail实例是通过JobBuilder类创建的，导入该类下的所有静态方法，会让你编码时有DSL的感觉：

```java
    import static org.quartz.JobBuilder.*;
```

让我们先看看Job的特征（nature）以及Job实例的生命期。不妨先回头看看教程一中的代码片段：

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

“HelloJob”类可以如下定义：

```java
public class HelloJob implements Job {

    public HelloJob() {
    }

    public void execute(JobExecutionContext context)
        throws JobExecutionException
    {
        System.err.println("Hello!  HelloJob is executing.");
    }
}
```

可以看到，我们传给scheduler一个JobDetail实例，因为我们在创建JobDetail时，将要执行的job的类名传给了JobDetail，所以scheduler就知道了要执行何种类型的job；每次当scheduler执行job时，在调用其execute(…)方法之前会创建该类的一个新的实例；执行完毕，对该实例的引用就被丢弃了，实例会被垃圾回收；这种执行策略带来的一个后果是，job必须有一个无参的构造函数（当使用默认的JobFactory时）；另一个后果是，在job类中，不应该定义有状态的数据属性，因为在job的多次执行中，这些属性的值不会保留。

那么如何给job实例增加属性或配置呢？如何在job的多次执行中，跟踪job的状态呢？答案就是:JobDataMap，JobDetail对象的一部分。

## JobDataMap

JobDataMap中可以包含不限量的（序列化的）数据对象，在job实例执行的时候，可以使用其中的数据；JobDataMap是Java Map接口的一个实现，额外增加了一些便于存取基本类型的数据的方法。

将job加入到scheduler之前，在构建JobDetail时，可以将数据放入JobDataMap，如下示例：

```java
JobDetail job = newJob(DumbJob.class)
    .withIdentity("myJob", "group1") // name "myJob", group "group1"
    .usingJobData("jobSays", "Hello World!")
    .usingJobData("myFloatValue", 3.141f)
    .build();
在job的执行过程中，可以从JobDataMap中取出数据，如下示例：

public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
        throws JobExecutionException
    {
        JobKey key = context.getJobDetail().getKey();

        JobDataMap dataMap = context.getJobDetail().getJobDataMap();

        String jobSays = dataMap.getString("jobSays");
        float myFloatValue = dataMap.getFloat("myFloatValue");

        System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
}
```

如果你使用的是持久化的存储机制（本教程的JobStore部分会讲到），在决定JobDataMap中存放什么数据的时候需要小心，因为JobDataMap中存储的对象都会被序列化，因此很可能会导致类的版本不一致的问题；Java的标准类型都很安全，如果你已经有了一个类的序列化后的实例，某个时候，别人修改了该类的定义，此时你需要确保对类的修改没有破坏兼容性；更多细节，参考现实中的序列化问题。另外，你也可以配置JDBC-JobStore和JobDataMap，使得map中仅允许存储基本类型和String类型的数据，这样可以避免后续的序列化问题。

如果你在job类中，为JobDataMap中存储的数据的key增加set方法（如在上面示例中，增加setJobSays(String val)方法），那么Quartz的默认JobFactory实现在job被实例化的时候会自动调用这些set方法，这样你就不需要在execute()方法中显式地从map中取数据了。

在Job执行时，JobExecutionContext中的JobDataMap为我们提供了很多的便利。它是JobDetail中的JobDataMap和Trigger中的JobDataMap的并集，但是如果存在相同的数据，则后者会覆盖前者的值。

下面的示例，在job执行时，从JobExecutionContext中获取合并后的JobDataMap：

```java
public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
        throws JobExecutionException
    {
        JobKey key = context.getJobDetail().getKey();

        JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

        String jobSays = dataMap.getString("jobSays");
        float myFloatValue = dataMap.getFloat("myFloatValue");
        ArrayList state = (ArrayList)dataMap.get("myStateData");
        state.add(new Date());

        System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
}
```

如果你希望使用JobFactory实现数据的自动“注入”，则示例代码为：

```java
public class DumbJob implements Job {

    String jobSays;
    float myFloatValue;
    ArrayList state;

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
        throws JobExecutionException
    {
        JobKey key = context.getJobDetail().getKey();

        JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

        state.add(new Date());

        System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }

    public void setJobSays(String jobSays) {
        this.jobSays = jobSays;
    }

    public void setMyFloatValue(float myFloatValue) {
        myFloatValue = myFloatValue;
    }

    public void setState(ArrayList state) {
        state = state;
    }

}
```

你也许发现，整体上看代码更多了，但是execute()方法中的代码更简洁了。而且，虽然代码更多了，但如果你的IDE可以自动生成setter方法，你就不需要写代码调用相应的方法从JobDataMap中获取数据了，所以你实际需要编写的代码更少了。当前，如何选择，由你决定。

## Job实例

很多用户对于Job实例到底由什么构成感到很迷惑。我们在这里解释一下，并在接下来的小节介绍job状态和并发。

你可以只创建一个job类，然后创建多个与该job关联的JobDetail实例，每一个实例都有自己的属性集和JobDataMap，最后，将所有的实例都加到scheduler中。

比如，你创建了一个实现Job接口的类“SalesReportJob”。该job需要一个参数（通过JobdataMap传入），表示负责该销售报告的销售员的名字。因此，你可以创建该job的多个实例（JobDetail），比如“SalesReportForJoe”、“SalesReportForMike”，将“joe”和“mike”作为JobDataMap的数据传给对应的job实例。

当一个trigger被触发时，与之关联的JobDetail实例会被加载，JobDetail引用的job类通过配置在Scheduler上的JobFactory进行初始化。默认的JobFactory实现，仅仅是调用job类的newInstance()方法，然后尝试调用JobDataMap中的key的setter方法。你也可以创建自己的JobFactory实现，比如让你的IOC或DI容器可以创建/初始化job实例。

在Quartz的描述语言中，我们将保存后的JobDetail称为“job定义”或者“JobDetail实例”,将一个正在执行的job称为“job实例”或者“job定义的实例”。当我们使用“job”时，一般指代的是job定义，或者JobDetail；当我们提到实现Job接口的类时，通常使用“job类”。

## Job状态与并发

关于job的状态数据（即JobDataMap）和并发性，还有一些地方需要注意。在job类上可以加入一些注解，这些注解会影响job的状态和并发性。

`@DisallowConcurrentExecution`：将该注解加到job类上，告诉Quartz不要并发地执行同一个job定义（这里指特定的job类）的多个实例。请注意这里的用词。拿前一小节的例子来说，如果“SalesReportJob”类上有该注解，则同一时刻仅允许执行一个“SalesReportForJoe”实例，但可以并发地执行“SalesReportForMike”类的一个实例。所以该限制是针对JobDetail的，而不是job类的。但是我们认为（在设计Quartz的时候）应该将该注解放在job类上，因为job类的改变经常会导致其行为发生变化。

`@PersistJobDataAfterExecution`：将该注解加在job类上，告诉Quartz在成功执行了job类的execute方法后（没有发生任何异常），更新JobDetail中JobDataMap的数据，使得该job（即JobDetail）在下一次执行的时候，JobDataMap中是更新后的数据，而不是更新前的旧数据。和 @DisallowConcurrentExecution注解一样，尽管注解是加在job类上的，但其限制作用是针对job实例的，而不是job类的。由job类来承载注解，是因为job类的内容经常会影响其行为状态（比如，job类的execute方法需要显式地“理解”其”状态“）。

如果你使用了@PersistJobDataAfterExecution注解，我们强烈建议你同时使用@DisallowConcurrentExecution注解，因为当同一个job（JobDetail）的两个实例被并发执行时，由于竞争，JobDataMap中存储的数据很可能是不确定的。

## Job的其它特性

通过JobDetail对象，可以给job实例配置的其它属性有：

Durability：如果一个job是非持久的，当没有活跃的trigger与之关联的时候，会被自动地从scheduler中删除。也就是说，非持久的job的生命期是由trigger的存在与否决定的；
RequestsRecovery：如果一个job是可恢复的，并且在其执行的时候，scheduler发生硬关闭（hard shutdown)（比如运行的进程崩溃了，或者关机了），则当scheduler重新启动的时候，该job会被重新执行。此时，该job的JobExecutionContext.isRecovering() 返回true。

## JobExecutionException

最后，是关于Job.execute(..)方法的一些其它细节。execute方法中仅允许抛出一种类型的异常（包括RuntimeExceptions），即JobExecutionException。因此，你应该将execute方法中的所有内容都放到一个”try-catch”块中。可以看看JobExecutionException的文档，以便更好地处理异常。

- 原文链接：[Lesson 3: More About Jobs and Job Details](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-03.html)
- 有兴趣的朋友可以参考我在github上的项目：[quartz源码注释](https://github.com/nkcoder/quartz-explained)