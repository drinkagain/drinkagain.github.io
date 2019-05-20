---
title: (13) SpringBoot 动手写一个Start
date: 2019-05-20 10:27:40
tags: SpringBoot
---
我们在使用SpringBoot 项目时，引入一个springboot start依赖，只需要很少的代码，或者不用任何代码就能直接使用默认配置，再也不用那些繁琐的配置了，感觉特别神奇。我们自己也动手写一个start.

### 一、新建一个 Start 的 Maven 项目 
<!--more-->
1. pom 文件如下
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.6</version>
        <optional>true</optional>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```
- spring-boot-autoconfigure springboot 自动配置的核心依赖
- spring-boot-starter-test 测试包
- lombok 省去 getter/setter 等简化代码

2. 演示代码

DemoService:
```java
public interface DemoService {

    String getMessage();

    Integer getCode();
}

```
DemoServiceImpl:    
```java
public class DemoServiceImpl implements DemoService {

    @Override
    public String getMessage() {
        return "Hello!";
    }

    @Override
    public Integer getCode() {
        return 123;
    }
}
```

DemoAutoConfiguration:
```java
@Configuration
public class DemoAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(DemoService.class)
    public DemoService demoService() {
        return new DemoServiceImpl();
    }
}
```
- @Configuration 标注该类为一个配置类
- ConditionalOnMissingBean(DemoService.class) 条件注解

>spingboot 的自动注解主要还是用这些条件注解来实现的。请查看之前的文章：

>[Spring Boot 自动配置之条件注解](https://juejin.im/post/5c6c2189e51d45713911466d)

>[Spring Boot 自动配置之@Enable*与@Import注解](https://juejin.im/post/5c761c096fb9a049b41d2299)

>[Spring Boot 自动配置之@EnableAutoConfiguration](https://juejin.im/post/5c7c7d4df265da2dcc800e87)

3. 让springboot 识别自动自动配置的代码
 
需要在resources下新建文件META-INF/spring.factories

```text
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.jiuxian.DemoAutoConfiguration
```

SpringBoot 中的注解 @EnableAutoConfiguration 在项目启动的时候会通过 SpringFactoriesLoader.loadFactoryNames 方法获取 spring.factories 文件下的配置类

4. 测试类
- 首先需要再包下建 Application run 的main 方法

```java
@SpringBootApplication
public class StartDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(StartDemoApplication.class, args);
    }
}

```

- 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class StartDemoApplicationTests {

    @Resource
    private DemoService demoService;

    @Test
    public void test() {
        String message = demoService.getMessage();
        System.out.println(message);
        Assert.assertEquals("Hello!", message);

        Integer code = demoService.getCode();
        System.out.println(code);
        Assert.assertEquals(123, (int) code);
    }
}
```

> 如果没有 StartDemoApplication 这个类则测试类启动的时候会报 @SpringBootApplication 找不到错误


### 二、新建 springboot 项目引入刚写的start项目

1. service

```java
@Service
public class TestService {

    @Resource
    private DemoService demoService;

    public void message() {
        System.out.println("code:" + demoService.getCode());
        System.out.println("message:" + demoService.getMessage());
    }
}

```

2. 测试
```java
    @Resource
    private TestService testService;

    @Test
    public void test() {
        testService.message();
    }

```

结果：
```text
code:123
message:Hello!
```

3. 重写DemoService方法
```java
@Service
public class DemoServiceImpl implements DemoService {

    @Override
    public String getMessage() {
        return "Hello!";
    }

    @Override
    public Integer getCode() {
        return 123;
    }
}

```
4. 测试结果
```text
code:123
message:Hello!
```

之所以这样的结果，是因为在start项目中的DemoService 实现类中有一个 @ConditionalOnMissingBean(DemoService.class) 的注解，如果不存在则使用默认的

### 三、Demo 源码

[GitHub源码地址](https://github.com/Zejun-Liu/springboot-start-demo.git)