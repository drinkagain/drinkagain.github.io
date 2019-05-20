---
title: (03) SpringBoot 对Future模式的支持
date: 2019-05-20 10:17:57
tags: SpringBoot
---

&emsp;&emsp;我们在实际项目中有些复杂运算、耗时操作，就可以利用多线程来充分利用CPU，提高系统吞吐量。SpringBoot对多线程支持非常好，对我们的开发非常便捷。<br/>
&emsp;&emsp;Future模式是多线程开发中非常常见的一种设计模式。核心思想是异步调用。当我们执行一个方法时，方法中有多个耗时任务需要同时去做，而且又不着急等待这个结果时可以让客户端立即返回然后，后台慢慢去计算任务。<br/>
&emsp;&emsp;当我们做一件事的时候需要等待，那么我们就可以在这个等待时间内来去做其它事情，这样就可以充分利用时间。比如我们点外卖，需要一段时间，那么我们在等外卖的时间里可以看点书，看个电影。这就是典型的Future模式。如果是普通模式的话，就是等外卖的时候就等外卖，外卖到了后再去看书，极大的浪费时间。<br/>
&emsp;&emsp;SpringBoot对Future模式支持非常好，只需要简单的代码就能实现。
<!--more-->
### 1.Future的相关方法
- boolean cancel(boolean mayInterruptIfRunning);
    //可以在任务执行过程中取消任务
- boolean isCancelled();
    //判断Future任务是否取消
- boolean isDone();
    //判断任务是否完成
- V get();//获取任务最终结果，这是一个阻塞方法，会等待任务执行好才会执行后面的代码 
- V get(long timeout, TimeUnit unit);
 //有等待时常的get方法，等待时间到了后仍然没有计算完成，则抛异常

### 2.需要的注解
 &emsp;springboot 配置多线程需要两个注解
1. @EnableAsync<br>
    在配置类中通过加@EnableAsync开启对异步任务的支持
2. @Async<br>
    在需要执行的方法上加@Async表明该方法是个异步方法，如果加在类级别上，则表明类所有的方法都是异步方法

### 3.配置代码
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //核心线程数
        taskExecutor.setCorePoolSize(8);
        //最大线程数
        taskExecutor.setMaxPoolSize(16);
        //队列大小
        taskExecutor.setQueueCapacity(100);
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```
### 4.FutureService
```java
@Service
public class FutureService {

    @Async
    public Future<String> futureTest() throws InterruptedException {
        System.out.println("任务执行开始,需要：1000ms");
        for (int i = 0; i < 10; i++) {
            Thread.sleep(100);
            System.out.println("do:" + i);
        }
        System.out.println("完成任务");
        return new AsyncResult<>(Thread.currentThread().getName());
    }
}
```
>【注】这里的方法自动被注入使用上文配置的ThreadPoolTaskExecutor

### 5.测试代码
```java
@Resource
private FutureService futureService;

@Test
public void futureTest() throws InterruptedException, ExecutionException {
    long start = System.currentTimeMillis();
    System.out.println("开始");
    //耗时任务
    Future<String> future = futureService.futureTest();
    //另外一个耗时任务
    Thread.sleep(500);
    System.out.println("另外一个耗时任务，需要500ms");

    String s = future.get();
    System.out.println("计算结果输出:" + s);
    System.out.println("共耗时：" + (System.currentTimeMillis() - start));
}
```
### 6.运行结果
```string
开始
2019-01-07 23:50:34.726  INFO 14648 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService
任务执行开始,需要：1000ms
do:0
do:1
do:2
do:3
另外一个耗时任务，需要500ms
do:4
do:5
do:6
do:7
do:8
do:9
完成任务
计算结果输出:ThreadPoolTaskExecutor-1
共耗时：1016

Process finished with exit code 0

```
**本来需要至少1500ms 执行的任务现在只需要1016ms,
因为在执行耗时任务1的同时也在执行耗时任务2，两个任务并行执行，这就是future模式的好处，在等待时间内去执行其它任务，能够充分利用时间**

>【注】本文基于SpringBoot 2.0

[GitHub 连接](https://github.com/drinkagain/SpringBoot/blob/master/springboot-future/Readme.md)

<br>
