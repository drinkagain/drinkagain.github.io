---
title: (09) SpringBoot 之条件注解
date: 2019-05-20 10:22:55
tags: SpringBoot
---
Spring Boot 神奇的自动配置，主要依靠大量的条件注解来使用配置自动化。

根据满足某一个特定条件创建一个特定的Bean。比如说，在某些系统变量下创建Bean，或者只有在某个Bean创建后才去创建另外一个Bean. 就是根据条件来控制Bean的创建行为，可以利用该特性来进行一些自动配置。

### 一、常用的条件注解
- @Conditional 依赖的条件
- @ConditionalOnBean  在某个Bean存在的条件下
- @ConditionalOnMissingBean 在某个Bean不存在的条件下
- @ConditionalOnClass  在某个Class存在的条件下
- @ConditionalOnMissingClass  在某个Class不存在的条件下
<!--more-->
比较常见的是这些注解，还有其它的比如
@ConditionalOnWebApplication,
@ConditionalOnProperty 等，可举一反三

### 二、特别说明 @Conditional 注解
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition Conditions} that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```

使用@Conditional注解，对象需要实现Condition接口，Condition 接口是一个函数式接口

```java
@FunctionalInterface
public interface Condition {

	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}

```

### 三、条件注解示例

示例场景：项目中动态的配置Mysql或者Oracle数据源

#### 1. 定义配置文件
``` text
db-type=oracle
```
#### 2. 定义Condition类
MySqlCondition.java
```java
public class MySqlCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return "mysql".equals(context.getEnvironment().getProperty("db-type"));
    }
}
```
OracleCondition.java
```java
public class OracleCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return "oracle".equals(context.getEnvironment().getProperty("db-type"));
    }
}
```
> 获取配置文件db-type的值

#### 3. JdbcFactory接口
```java
public interface JdbcFactory {

    void create();
}
```
#### 4. 默认的Mysql和Oracle实现
Mysql
```java
@ConditionalOnMissingBean(value = JdbcFactory.class, ignored = MySqlDefaultFactory.class)
@Conditional(MySqlCondition.class)
@Component
public class MySqlDefaultFactory implements JdbcFactory {

    @Override
    public void create() {
        System.out.println("Default MySql create ..");
    }

}

```
Oracle

```java
@ConditionalOnMissingBean(value = JdbcFactory.class, ignored = OracleDefaultFactory.class)
@Conditional(OracleCondition.class)
@Component
public class OracleDefaultFactory implements JdbcFactory {

    @Override
    public void create() {
        System.out.println("Default oracle create..");
    }
}
```
#### 5. 测试默认实现方式

```java
@Resource
private JdbcFactory jdbcFactory;

@Test
public void conditionOnMissBean() {
    jdbcFactory.create();
}
```

结果：
```text
Default MySql create ..
```
#### 6. 自定义实现方式
```java
@Component
public class MysqlFactory implements JdbcFactory {

    @Override
    public void create() {
        System.out.println("mysql 。。 create");
    }
}
```
#### 7. 测试
```java
@Resource
private JdbcFactory jdbcFactory;

@Test
public void conditionOnMissBean() {
    jdbcFactory.create();
}
```
结果：
```text
mysql 。。 create
```

#### 8.解析
> 当环境中不存在 JdbcFactory 的Bean时则使用默认的实现的方式，如例：没有自定义实现时，则默认使用MySqlDefaultFactory。这在自动化配置中会经常用到。比如redisTemplate 的默认实现

### 四、GitHub源码
[源码地址](https://github.com/Zejun-Liu/SpringBoot2.0/tree/master/springboot-annotations)
