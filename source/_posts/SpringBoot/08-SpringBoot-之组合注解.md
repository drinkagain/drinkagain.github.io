---
title: (08) SpringBoot 之组合注解
date: 2019-05-20 10:22:25
tags: SpringBoot
---
SpringBoot应用开发，会大量的使用注解，有些注解会经常一起使用，如果能通过一个组合注解进行包装则能够简化代码，并且还会避免因为少了某些注解而报错

### 一、 常见的组合注解
#### 1. @SpringBootApplication
该注解是SpringBoot项目的核心注解，该注解包含：
-  @SpringBootConfiguration
-  @EnableAutoConfiguration
-  @ComponentScan
<!--more-->
@SpringBootApplication 注解就有了自动配置功能 、扫描包功能。

@EnableAutoConfiguration 让SpringBoot根据类路径中的jar包依赖为当前项目进行自动配置。例如，添加spring-boot-starter-web依赖，会自动添加tomcat和SpringMVC的依赖，SpringBoot 会对Tomcat和SpringMVC进行自动配置

@ComponentScan 会自动扫描@SpringBootApplication所在类的同级包以及子包的Bean。所以建议入口类放在groupId+artifactId组合下，或者groupId下。

在SpringBoot项目启动类上用这三个注解替换@SpringBootApplication也是可以的

#### 2. @Configuration
该注解包含@Component注解，该注解不单标注该类是一个配置类，而且声明该类是一个Bean

#### 3. @Enable*
@Enable* 类的注解都有一个@Import注解，该注解是用来导入配置类的，其实就是导入了一些自动配置的Bean，有以下三类：

1. 直接导入配置类

    导入一个有 @Configuration的Bean
2. 依据条件选择配置类

    导入一个实现了ImportSelector接口的配置类
3. 动态注册Bean

    导入一个实现了ImportBeanDefinitionRegistrar接口的配置类
    
> **本文不做深入探讨，会另出一篇关于@Import的使用**

### 二、自定义组合注解
我们在配置类上加@ComponentScan时还会写@Configuration我们可以写一个组合注解

1. 组合注解
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@ComponentScan
public @interface ComponentScanConfig {
    
    //这个必须写，覆盖@ComponentScan的注解value的值
    String[] value() default {};
}

```
> 【注】String[] value() default {}; 是为了覆盖@ComponentScan的注解value的值

2. service
```java
public class CombinationAnnotationTestService {

    public void doSth() {
        System.out.println("doSth....");
    }
}
```

3. 配置类
```java
@ComponentScanConfig("com.jiuxian.combination")
public class CombinationAnnotationConfig {

    @Bean
    public CombinationAnnotationTestService combinationTestService() {
        return new CombinationAnnotationTestService();
    }
}
```

4. 测试

(1)
```java
public class Main {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(CombinationAnnotationConfig.class);
        CombinationAnnotationTestService service = context.getBean(CombinationAnnotationTestService.class);
        service.doSth();
        context.close();
    }
}
```
(2)
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootAnnotationsApplicationTests {

    @Resource
    private CombinationAnnotationTestService combinationAnnotationTestService;

    @Test
    public void combinationTest() {
        combinationAnnotationTestService.doSth();
    }

}
```

5. 结果
```string
doSth....
```

### 三、GitHub源码
[GitHub源码](https://github.com/Zejun-Liu/SpringBoot2.0/tree/master/springboot-annotations)


