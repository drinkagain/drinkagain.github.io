---
title: (04) SpringBoot 定时任务Scheduled
date: 2019-05-20 10:19:14
tags: SpringBoot
---
&emsp;&emsp;SpringBoot定时任务使用@EnableScheduling和@Scheduled这两个注解就能够简单实现，在集群环境下建议用Quartz等实现。<br>
&emsp;&emsp;不多说看代码具体实现
### 1.首先开启对Scheduled的支持
<!--more-->
```java
@Configuration
@EnableScheduling
public class ScheduledConfig {
}
```
### 2.使用@Scheduled注解
```java

@Service
public class ScheduleService {

    @Scheduled(fixedDelay = 2000)
    public void scheduleTest1() throws InterruptedException {
        System.out.println("scheduleTest1 Start.>>" + new Date().toLocaleString());
        Thread.sleep(6000);
        System.out.println("scheduleTest1 End.>>" + new Date().toLocaleString());
    }

    @Scheduled(fixedRate = 2000)
    public void scheduleTest2() throws InterruptedException {
        System.out.println("scheduleTest2 Start.>>" + new Date().toLocaleString());
        Thread.sleep(6000);
        System.out.println("scheduleTest2  End.>>");
    }

    @Scheduled(cron = "0 0/1 * * * ? ")
    public void scheduleTest3() {
        System.out.println("scheduleTest3 >>>");
    }

    @Scheduled(fixedRate = 2000, initialDelay = 1000)
    public void scheduleTest4() throws InterruptedException {
        System.out.println("scheduleTest2 fixedRate Start.>>");
        Thread.sleep(6000);
        System.out.println("scheduleTest2 fixedRate End.>>");
    }

}

```
### 3.Scheduled注解中参数解释
- fixedDelay,fixedDelayString 都表式上次任务完成之后，多少毫秒后开始下一个任务
  + fixedDelay 传毫秒数
  + fixedDelayString 毫秒的String值或者java.time.Duration 类型想匹配的时间 比如PT20.345S
- fixedRate,fixedRateString 表式上次执行之后多少毫秒后开始下一个任务
  + fixedRate 传毫秒数
  + fixedRateString 毫秒的String值或者java.time.Duration 类型想匹配的时间
- initialDelay,initialDelayString 延迟多少毫秒执行
- cron 使用cron表达式来表示

### 4.那些坑
1. 首行启动之后他会自动查找org.springframework.scheduling.TaskScheduler的Bean或者是
  或者名为“taskScheduler”的Bean或者 java.util.concurrent.ScheduledExecutorService的Bean 如果都找不到将会以本地单线程的方式执行。你会发现定时任务会一个执行完成之后才会执行下一个。
2. 如果是简单的通过@EnableAsync 然后再方法上注解@Async后，则fixedDelay/fixedDelayString参数将失效等同于fixedRate/fixedRateString,因为方法上加@Async注解之后等同于该方法为异步方法，不会等待任务完成
### 5.配置多线程执行

1. 
```java
    @Bean
    public ScheduledExecutorService scheduledExecutorService() {
        return new ScheduledThreadPoolExecutor(10);
    }
 ```
 2.
 ```java
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
        threadPoolTaskScheduler.setPoolSize(20);
        return threadPoolTaskScheduler;
    }
 ```
 3.
 ```java
@Configuration
@EnableScheduling
public class ScheduledConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
    }

    @Bean(destroyMethod = "shutdown")
    public Executor taskExecutor() {
        return Executors.newScheduledThreadPool(20);
    }
}
 ```
 > 【注】代码基于SpringBoot 2.0
 
[GitHub源码](https://github.com/Zejun-Liu/SpringBoot/blob/master/springboot-schedule/Readme.md)
