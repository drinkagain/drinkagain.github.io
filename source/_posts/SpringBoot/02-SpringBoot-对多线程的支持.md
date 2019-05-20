---
title: (2) SpringBoot 对多线程的支持
date: 2019-05-20 10:16:11
tags: SpringBoot
---
&emsp;&emsp;我们在实际项目中有些复杂运算、耗时操作，就可以利用多线程来充分利用CPU，提高系统吞吐量。SpringBoot对多线程支持非常好，对我们的开发非常便捷。<br>


### 1.需要的注解
 &emsp;springboot 配置多线程需要两个注解
 <!--more-->
1. @EnableAsync<br>
    在配置类中通过加@EnableAsync开启对异步任务的支持
2. @Async<br>
    在需要执行的方法上加@Async表明该方法是个异步方法，如果加在类级别上，则表明类所有的方法都是异步方法
### 2.配置代码
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
### 3.Service
```java
@Service
public class AsyncService {

    @Async
    public void executeAsync1() {
        Thread.sleep(20);
        System.out.println("异步任务::1");

    }

    @Async
    public void executeAsync2() {
        System.out.println("异步任务::2");
    }

}

```
>【注】这里的方法自动被注入使用上文配置的ThreadPoolTaskExecutor
### 4.测试代码
```java
    @Resource
    private AsyncService asyncService;

    @Test
    public void asyncTest() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            asyncService.executeAsync1();
            asyncService.executeAsync2();
        }
        Thread.sleep(1000);
    }
    
```
### 5.运行结果
```string
异步任务::2
异步任务::2
异步任务::2
异步任务::2
异步任务::2
异步任务::2
异步任务::2
异步任务::1
异步任务::1
异步任务::1
异步任务::1
异步任务::1
异步任务::1
异步任务::1
异步任务::1
异步任务::2
异步任务::2
异步任务::2
异步任务::1
异步任务::1
```


>【注】本文基于SpringBoot 2.0

[GitHub 连接](https://github.com/drinkagain/SpringBoot/blob/master/springboot-future/Readme.md)

<br>
感谢《Spring Boot实战 JavaEE开发的颠覆者》
