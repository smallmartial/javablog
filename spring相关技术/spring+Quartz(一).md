## 1.Quartz大致介绍

### 1.1介绍

Quartz是OpenSymphony开源组织在Job scheduling领域又一个开源项目，是完全由java开发的一个开源的任务日程管理系统，“任务进度管理器”就是一个在预先确定（被纳入日程）的时间到达时，负责执行（或者通知）其他软件组件的系统。
Quartz用一个小Java库发布文件（.jar文件），这个库文件包含了所有Quartz核心功能。这些功能的主要接口(API)是Scheduler接口。它提供了简单的操作，例如：将任务纳入日程或者从日程中取消，开始/停止/暂停日程进度。

### 1.2 Quartz任务调度主要元素

- Quartz任务调度的主要元素有：

  Trigger(触发器)

  Scheduler(任务调度器)

  Job(任务)

其中Trigger，Job是元数据，Scheduler才是任务调度的控制器。

![image-20200418203736062](spring+Quartz(%E4%B8%80)/image-20200418203736062.png)

### 1.3 Quartz特点

> 1. 强大的调度功能，例如支持多样的调度方式
> 2. 灵活的应用方式，例如支持任务和调度的多种组合方式
> 3. 分布式和集群功能，在被Terracotta收购后，在Quartz的基础上的拓展

## 2.定时器种类

Quartz 中五种类型的 Trigger：SimpleTrigger，CronTirgger，DateIntervalTrigger，NthIncludedDayTrigger和Calendar 类（ org.quartz.Calendar）。
最常用的：
SimpleTrigger：用来触发只需执行一次或者在给定时间触发并且重复N次且每次执行延迟一定时间的任务。
CronTrigger：按照日历触发，例如“每个周五”，每个月10日中午或者10：15分。

## 3.重要组成

任务： 

- Job：表示一个工作，要执行的具体内容。此接口中只有一个方法。要创建一个任务，必须得实现这个接口。该接口只有一个execute方法，任务每次被调用的时候都会执行这个execute方法的逻辑，类似TimerTask的run方法，在里面编写业务逻辑。 

  ```
    public class TestJob implements Job {
        /**把要执行的操作，写在execute方法中  */
        @Override
        public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
            System.out.println("I can do something...");
            System.out.println(sdf.format(new Date()));
        }
    }复制代码
  ```

生命周期：在每次调度器执行job时，它在调用execute方法前会创建一个新的job实例，当调用完成之后，关联的job对象实例会被释放，释放的实例会被垃圾回收机制回收。

- JobBuilder：可向任务传递数据,通常情况下,我们使用它就可向任务类发送数据了，如有特别复杂的传递参数,它提供了一个传递递:JobDataMap对象的方法　　

  ```
    JobDetail jobDetail =  JobBuilder.newJob(TestJob.class).withIdentity("testJob","group1").build();复制代码
  ```

- JobDetail：用来保存我们任务的详细信息。一个JobDetail可以有多个Trigger，但是一个Trigger只能对应一个JobDetail。下面是JobDetail的一些常用的属性和含义：

- JobStore：负责跟踪所有你给scheduler的“工作数据”：jobs, triggers, calendars, 等。

  - RAMJobStore：是使用最简单的也是最高效(依据CPU时间)的JobStore 。RAMJobStore 正如它名字描述的一样，它保存数据在RAM。缺点是你的应用结束之后所有的数据也丢失了--这意味着RAMJobStore 不具有保持job和trigger持久的能力。对于一些程序是可以接受的，甚至是期望的，但对于其他的程序可能是灾难性的。使用RAMJobStore配置Quartz：配置如下

    ```
      org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore 复制代码
    ```

  - JDBCJobStore：以JDBC的方式保存数据在数据库中。它比RAMJobStore的配置复杂一点，也没有RAMJobStore快。然而,性能缺点不是糟透了,特别是如果你在数据库表主键上建立了索引。在机器之间的LAN(在scheduler 和数据库之间)合理的情况下，检索和更新一个被触发的Trigger花费的时间少于10毫秒。几乎适用于所有的数据库，广泛用于 Oracle。PostgreSQL, MySQL, MS SQLServer, HSQLDB, 和DB2。使用JDBCJobStore之前你必须首先创建一系列Quartz要使用的表。你可以发现表创建语句在Quartz发布目录的 “docs/dbTables”下面。你需要确定你的应用要使用的事务类型。如果你不想绑定调度命令(例如增加和移除Trigger)到其他的事务，你可以使用JobStoreTX (最常用的选择)作为你的Jobstore。如果你需要Quartz和其他的事务(例如在J2EE应用服务器中)一起工作，你应该使用JobStoreCMT ，Quartz 将让应用服务器容器管理这个事务。使用JobStoreTx配置Quartz：

    ```
      org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX  
      org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate  
      #配置表的前缀  
      org.quartz.jobStore.tablePrefix = QRTZ_  
      #使用JNDI数据源的时候，数据源的名字  
      org.quartz.jobStore.dataSource = myDS      复制代码
    ```

  - TerracottaJobStore：提供了一个方法：在不使用数据库的情况下使它具有收缩性和强壮性。可以是集群的也可以是非集群的，在这两种情况下为你的job数据提供了一个存储机制用于应用程序重启之间持久,因为数据是存储在Terracotta服务器。它的性能比使用数据库访问JDBCJobStore好一点儿(大约是一个数量级)，但是明显比RAMJobStore慢。使用TerracottaJobStore配置Quartz：

    ```
      org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore  
      org.quartz.jobStore.tcConfigUrl = localhost:9510  复制代码
    ```

  

- JobdataMap：中可以包含不限量的（序列化的）数据对象，在job实例执行的时候，可以使用其中的数据；JobDataMap是Java Map接口的一个实现，额外增加了一些便于存取基本类型的数据的方法。

- 2）触发器：用来触发执行Job

![image-20200418221532984](spring+Quartz(%E4%B8%80)/image-20200418221532984.png)

- Cron表达式：用于配置CronTrigger实例，是由7个表达式组成的字符串，描述了时间表的详细信息。( 利用工具，在线生成cron表达式：[cron.qqe2.com/](http://cron.qqe2.com/))

  格式为：[秒][分][时][日][月][周][年]
  　　Cron表达式特殊字符意义对应表：

![image-20200418222121395](spring+Quartz(%E4%B8%80)/image-20200418222121395.png)

- NthIncludedDayTrigger：是 Quartz 开发团队最新加入到框架中的一个 Trigger。它设计用于在每一间隔类型的第几天执行 Job。例如，你要在每个月的 15 号执行开票的 Job，用 NthIncludedDayTrigger就再合适不过了。

  ```java
    NthIncludedDayTrigger trigger = new NthIncludedDayTrigger("NthIncludedDayTrigger",Scheduler.DEFAULT_GROUP);
                trigger.setN(15);
                trigger.setIntervalType(NthIncludedDayTrigger.INTERVAL_TYPE_MONTHLY);
  
  ```

- `Scheduler` ：这是Quartz Scheduler的主要接口，代表一个独立运行容器。调度程序维护JobDetails和触发器的注册表。 一旦注册，调度程序负责执行作业，当他们的相关联的触发器触发（当他们的预定时间到达时）。

