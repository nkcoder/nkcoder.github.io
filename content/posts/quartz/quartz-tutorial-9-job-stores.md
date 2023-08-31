---
title: Quartz系列九：Job Stores
description: Quartz系列文章的第九篇：介绍Job Stores
date: 2020-02-19T21:12:59+08:00
categories:
- quartz
draft: false
---

JobStore 主要是追踪 scheduler 中的"工作数据": jobs, triggers, calendars 等。为 scheduler 选择合适的 JobStore 是很重要的一步。如果你理解了它们之间的区别，选择就很简单了。在 SchedulerFactory 使用的配置文件（或对象）中声明 scheduler 的 JobStore（通过配置）。

> 不要在代码中直接声明 JobStore 实例。由于某些原因，很多人尝试这样做。JobStore 是 Quartz 幕后使用的。你只需要通过配置告诉 Quartz 使用哪个 JobStore，然后在代码中使用 Scheduler 接口即可。

## RAMJobStore

RAMJobStore 是最容易使用的 JobStore，也是性能最好的（从 CPU 角度来看）。从名字就可以看出来：它将所有数据存在 RAM 中。这也是它如此快、如此易配置的原因。缺点是：当应用停止（或崩溃）时，所有的调度信息都将丢失 - 这也意味着 RAMJobStore 并不支持 jobs 和 triggers 的“持久性”。对有些应用来说这是可接受的 - 甚至是期望的，但是对于其它的应用，这可能是灾难性的。

**配置 Quartz 使用 RAMJobStore**

```java
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

没有其它的配置了。

## JDBCJobStore

JDBCJobStore - 将所有的数据通过 JDBC 保存在数据库中。正因如此，它比 RAMJobStore 的配置要复杂一些，也没那么快。但是，它的性能缺陷并没有那么不堪，尤其是当你在数据库的主键上都创建了索引。在目前主流的机器上，在局域网（scheduler 和 databasae 之间）环境下，接收和更新一个 trigger 的时间一般小于 10 毫秒。

JDBCJobStore 可以与几乎所有的数据库一起工作，它已经被广泛用于 Oracle、PostgreSQL、MySQL、MS SQLServer、HSQLDB 和 DB2。要使用 JDBCJobStore，首先需要创建一组 Quartz 使用的数据库。你可以在 Quartz 包的"docs/dbTables"目录下找到创建表的 SQL 脚本。如果你没找到适合你的数据库的脚本，在其中一种脚本上修改下即可。需要注意的是：所有的表都是以"QRTZ\_"为前缀的（如"QRTZ_TRIGGERS"和“QRTZ_JOB_DETAIL”）。前缀可以是任意值，只要在配置文件中告诉 JDBCJobStore 前缀的值即可。在同一个数据库中，对于不同的 scheduler 实例，使用不同的前缀去创建表是非常有用的。

表创建好后，在配置和使用 JDBCJobStore 之前，还有重要的一步。你需要决定使用哪种事务。如果你不需要将调度命令（如增加和删除 trigger）与其它事务绑定，你可以让 Quartz 通过**JobStoreTX**来管理事务（这是最常用的选择）。

如果你需要 Quartz 与其它事务一起工作（比如在 J2EE 应用中），你应该使用**JobStoreCMT** - 在这种情况下，Quartz 会让应用服务器管理事务。

最后一步就是设置 DataSource，以便 JDBCJobStore 可以从数据库获取连接。DataSource 可以使用多种方式在 Quartz 配置文件中定义。一种方式是让 Quartz 自己创建和管理 DataSource - 通过提供数据库所有连接相关的信息。另一种方式是应用服务器提供 DataSource - 为 JDBCJobStore 提供 DataSource 的 JNDI 名称。配置细节，参考"docs/config"目录下的样例配置文件。

使用 JDBCJobStore（假设你当前使用的是 StdSchedulerFactory），你首先需要将 JobStore 的 class 属性设置为：org.quartz.impl.jdbcjobstore.JobStoreTX 或者 org.quartz.impl.jdbcjobstore.JobStoreCMT，取决于你选择的事务管理方式。

**Configuring Quartz to use JobStoreTx**

```java
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
```

然后你需要选择 JobStore 使用的 DriverDelegate。DriverDelegate 主要做数据库相关的 JDBC 工作。StdJDBCDelegate 是使用普通的 JDBC 代码（SQL 语句）的一种代理。如果没有与数据库特定的代理，可以尝试使用 StdJDBCDelegate - 我们仅为那些使用 StdJDBCDelegate 有问题的数据库提供了特定的代理。其它代理可以在“org.quartz.impl.jdbcjobstore”包或子包下找到，包括 DB2v6Delegate, HSQLDBDelegate, MSSQLDelegate, PostgreSQLDelegate,WeblogicDelegate, OracleDelegate 等。

选好了代理，将 class name 设置为 JDBCJobStore 使用的代理：

```java
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
```

接下来，需要告诉 JobStore 表名的前缀：

```java
org.quartz.jobStore.tablePrefix = QRTZ_
```

最后，设置 JobStore 使用的 DataSource。DataSource 的名字必须在 Quartz 配置文件中定义。比如我们使用名为“myDS”的 DataSource（在配置文件中已定义）：

**Configuring JDBCJobStore with the name of the DataSource to use**

```java
org.quartz.jobStore.dataSource = myDS
```

> 如果你的 Scheduler 很忙（比如，总是执行与线程池线程数量一致的 job），可能需要将 DataSource 中连接数的数量设置为线程池大小+2.

> 将"org.quartz.jobStore.useProperties"参数设置为“true"（默认是 false）告诉 JDBCJobStore，JobDataMaps 中所有的值都是 String 类型，因此可以使用 name-value 形式存储，而不是在 BLOB 列中存储复杂对象的序列化结果。从长远来讲更安全，因为避免了将非 String 了行序列化为 BLOG 过程中的版本问题。

## TerracottaJobStore

TerracottaJobStore 提供了更好的扩展性和健壮性，且不使用数据库。意味着你的数据库不需要支持 Quartz 的负载，可以将数据库资源用于应用的其它部分。

TerracottaJobStore 可以运行在集群或非集群模式，在任何一种模式下都可以保证在应用重启时 job 数据的持久化，因为数据是存储在 Terracotta 服务器上。性能比使用数据库高很多（差不多一个量级），但比 RAMJobStore 要慢。

要使用 TerracottaJobStore（假设你当前使用的是 StdSchedulerFactory），只需要设置 org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore，并添加额外一行配置指定 Terracotta 服务器的地址：

```java
org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore
org.quartz.jobStore.tcConfigUrl = localhost:9510
```

关于 JobStore 和 Terracotta 的更多信息请参考：http://www.terracotta.org/quartz 。

- 原文链接[Lesson 9: Job Stores](http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/tutorials/tutorial-lesson-09.html)
- 有兴趣的朋友可以参考我在 github 上的项目：[quartz 源码注释](https://github.com/nkcoder/quartz-explained)


