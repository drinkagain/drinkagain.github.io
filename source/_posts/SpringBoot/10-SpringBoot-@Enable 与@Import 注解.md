---
title: (10) SpringBoot @Enable* 与@ Import注解
date: 2019-05-20 10:23:59
tags: SpringBoot
---
SpringBoot 的自动配置如此强大，比如我们经常使用的@Enable\* 注解来开启对某方面的支持。那么@Enable\* 注解的原理是什么呢？

### 一、@Enable\* 注解与 @Import 注解之间的关系
<!--more-->
@Enable* 举例：
- @EnableScheduling 开启计划任务的支持
- @EnableAsync 开启异步方法的支持
- @EnableAspectJAutoProxy 开启对 AspectJ 代理的支持
- @EnableTransactionManagement 开启对事务的支持
- @EnableCaching 开启对注解式缓存的支持

......

我们观察这些@Enable\* 的源码可以看出，所有@Enable\* 注解都是有@Import的组合注解，@Enable* 自动开启的实现其实就是导入了一些自动配置的Bean

看下 Spring Boot Reference Guide原文

```
You need not put all your @Configuration into a single class. The @Import annotation
can be used to import additional configuration classes.

您不需要把所有的 @Configuration 放到一个类中。@Import 注解可以导入额外的配置类。
```

@Import 注解的最主要功能就是导入额外的配置信息

### 二、 @Import 注解的用法
官方介绍：
```
* <p>Provides functionality equivalent to the {@code <import/>} element in Spring XML.
 * Allows for importing {@code @Configuration} classes, {@link ImportSelector} and
 * {@link ImportBeanDefinitionRegistrar} implementations, as well as regular component
 * classes (as of 4.2; analogous to {@link AnnotationConfigApplicationContext#register}).
```
有以下三种使用方式
#### 1、直接导入配置类（@Configuration 类）
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}
```

可以看到EnableScheduling注解直接导入配置类SchedulingConfiguration，这个类注解了@Configuration，且注册了一个scheduledAnnotationProcessor的Bean，SchedulingConfiguration的源码如下：
```java
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class SchedulingConfiguration {

	@Bean(name = TaskManagementConfigUtils.SCHEDULED_ANNOTATION_PROCESSOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public ScheduledAnnotationBeanPostProcessor scheduledAnnotationProcessor() {
		return new ScheduledAnnotationBeanPostProcessor();
	}

}

```
#### 2、依据条件选择配置类（实现 ImportSelector 接口）
如果并不确定引入哪个配置类，需要根据@Import注解所标识的类或者另一个注解(通常是注解)里的定义信息选择配置类的话，用这种方式。

ImportSelector接口只有一个方法
```java
String[] selectImports(AnnotationMetadata importingClassMetadata);
```
AnnotationMetadata：用来获得当前配置类上的注解

例：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {

	Class<? extends Annotation> annotation() default Annotation.class;
	
	boolean proxyTargetClass() default false;

	AdviceMode mode() default AdviceMode.PROXY;

	int order() default Ordered.LOWEST_PRECEDENCE;

}

```
AsyncConfigurationSelector继承AdviceModeImportSelector，AdviceModeImportSelector类实现ImportSelector接口
根据AdviceMode的不同来选择生明不同的Bean
```java
public class AsyncConfigurationSelector extends AdviceModeImportSelector<EnableAsync> {

	private static final String ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME =
			"org.springframework.scheduling.aspectj.AspectJAsyncConfiguration";

	@Override
	@Nullable
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {ProxyAsyncConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}

}

```
#### 3、动态注册Bean（实现 ImportBeanDefinitionRegistrar 接口）
一般只要用户确切知道哪些Bean需要放入容器的话，自己可以通过spring 提供的注解来标识就可以了，比如@Component,@Service,@Repository,@Bean等。
如果是不确定的类，或者不是spring专用的，所以并不想用spring的注解进行侵入式标识，那么就可以通过@Import注解，实现ImportBeanDefinitionRegistrar接口来动态注册Bean。
比如：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	boolean proxyTargetClass() default false;
	
	boolean exposeProxy() default false;

}
```
AspectJAutoProxyRegistrar实现了ImportBeanDefinitionRegistrar接口，ImportBeanDefinitionRegistrar的作用是在运行时自动添加Bean到已有的配置类，通过重写方法：
```java
public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);
```
- AnnotationMetadata  参数用来获得当前配置类上的注解
- BeanDefinitionRegistry 参数用来注册Bean

源码：
```java
@Override
public void registerBeanDefinitions(
		AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

	AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

	AnnotationAttributes enableAspectJAutoProxy =
			AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
	if (enableAspectJAutoProxy != null) {
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}
}
```

Mybatis 中大名鼎鼎的@MapperScan 也是如此

### 三、官方文档
[官方文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-configuration-classes)
