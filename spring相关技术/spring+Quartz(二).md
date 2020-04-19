## 1.Quartz示例

```java

public class JobTest implements Job {
 
    /**
     * 具体执行的任务
     *
     * @param jobExecutionContext 保存着该job运行时的一些信息
     */
    @Override
    public void execute(JobExecutionContext jobExecutionContext) {
        //执行job的scheduler的引用
        Scheduler scheduler = jobExecutionContext.getScheduler();
        //触发job的trigger的引用
        Trigger trigger = jobExecutionContext.getTrigger();
        //JobDetail对象引用，以及一些其它信息
        JobDetail jobDetail = jobExecutionContext.getJobDetail();
        //获取参数
        //jobExecutionContext中的JobDataMap：他是JobDetail中的JobDataMap和Trigger中的JobDataMap的并集
        JobDataMap jobDataMap = jobExecutionContext.getMergedJobDataMap();
        Object jobDetailValue = jobDataMap.get("jobDetailKey");
        Object simpleTriggerValue = jobDataMap.get("simpleTriggerKey");
        Object cronTriggerValue = jobDataMap.get("cronTriggerKey");
        //任务执行详情
        String key = "jobDetailValue:" + jobDetailValue + " simpleTriggerValue:" + simpleTriggerValue + " cronTriggerValue:" + cronTriggerValue;
        System.err.println(new Date() + ": doing something...获取到的key:" + key);
    }
}
 
class QuartzTest {
    public static void main(String[] args) throws Exception {
        //1、创建JobDetial对象
//        每次当scheduler执行job时，在调用其execute(…)方法之前会创建JobTest类的一个新的实例
        JobDetail jobDetail = JobBuilder.newJob(JobTest.class)
                //设置JobDetial的分组和名称
                .usingJobData("jobDetailKey", "jobDetailValue")
                .withIdentity("job1", "group1")
                .build();
 
        //2、创建Trigger对象:SimpleTrigger
        Trigger simpleTrigger = newTrigger()
                //设置触发器的分组和名称
                .withIdentity("simpleTrigger", "simpleTriggerGroup")
                //开始时间
                .startAt(new Date())
                //结束时间
                .endAt(new Date())
                //设置优先级
                .withPriority(1)
                //设置参数
                .usingJobData("simpleTriggerKey", "simpleTriggerValue")
                .withSchedule(simpleSchedule()
                        //每隔一秒执行一次
                        .withIntervalInSeconds(1)
                        //一直执行
                        .repeatForever())
                .build();
 
        //创建Trigger对象:CronTrigger
        CronTrigger cronTrigger = newTrigger()
                .usingJobData("cronTriggerKey", "cronTriggerValue")
                .withIdentity("cronTrigger", "cronTriggerGroup")
                .withSchedule(CronScheduleBuilder.cronSchedule("0/1 * * * * ?"))
                //设置优先级
                .withPriority(1)
                .build();
 
        //3、创建调度器：Scheduler
        Scheduler scheduler = new StdSchedulerFactory().getScheduler();
        //配置JobDetail和Trigger对象
        scheduler.scheduleJob(jobDetail, cronTrigger);
        //4、并执行启动、关闭等操作
        scheduler.start();
        Thread.sleep(10000);
 
        //关闭调度器：是否等待job执行完成才关闭
        scheduler.shutdown(true);
        System.err.println("-------关闭调度器-------");
 
    }
}
```

## 2.Springboot集成quartz

### 2.1添加依赖

```pom
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-quartz</artifactId>
    </dependency>
```

### 2.2自动装配介绍

我们可以再spingboot的自动装配源码可以看到quartz定义了如下的接口和类：

![image-20200419132908352](spring+Quartz(%E4%BA%8C)/image-20200419132908352.png)

- `JobStoreType` 该类是一个枚举类型，定义了对应`application.yml`、`application.properties`文件内`spring.quartz.job-store-type`配置，其目的是配置`quartz`任务的数据存储方式，分别为：MEMORY（内存方式：`默认`）、JDBC（数据库方式）。

- `QuartzAutoConfiguration` 该类是自动配置的主类，内部配置了`SchedulerFactoryBean`以及`JdbcStoreTypeConfiguration`，使用`QuartzProperties`作为属性自动化配置条件。

- `QuartzDataSourceInitializer` 该类主要用于数据源初始化后的一些操作，根据不同平台类型的数据库进行选择不同的数据库脚本。

- `QuartzProperties` 该类对应了`spring.quartz`在`application.yml`、`application.properties`文件内开头的相关配置。

- `SchedulerFactoryBeanCustomizer` 这是一个接口，我们实现该接口后并且将实现类使用`Spring IOC`托管，可以完成`SchedulerFactoryBean`的个性化设置，这里的设置完全可以对`SchedulerFactoryBean`做出全部的设置变更。

### 2.3继承`QuartzJobBean`

继承`QuartzJobBean并重写`executeInternal`方法，与之前的实现`Job`接口类似

```java
/**
 * @Author smallmartial
 * @Date 2020/4/19
 * @Email smallmarital@qq.com
 */
public class SampleJob  extends QuartzJobBean {

    //作业可以定义设置器以注入数据映射属性。常规豆也可以类似的方式注入，如以下示例所示：
    @Autowired
    private TestService testService;
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        testService.test();
        System.out.println("    Hi! :" + jobExecutionContext.getJobDetail().getKey());
    }
}

```

这里`TestService`打印一条`uuid`模拟调用service的场景

### 2.4添加配置类

```JAVA
@Configuration
public class QuartzConfig {
    @Bean
    public JobDetail uploadTaskDetail() {
        return JobBuilder.newJob(SampleJob.class).withIdentity("mjtTask").storeDurably().build();
    }

    @Bean
    public Trigger uploadTaskTrigger() {
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("*/5 * * * * ?");
        return TriggerBuilder.newTrigger().forJob(uploadTaskDetail())
                .withIdentity("mjtTask")
                .withSchedule(scheduleBuilder)
                .build();
    }
}
```

## 2.5运行结果如下

![image-20200419150047206](spring+Quartz(%E4%BA%8C)/image-20200419150047206.png)