---
title: (11) SpringBoot @EnableAutoConfiguration
date: 2019-05-20 10:25:37
tags: SpringBoot
---
Spring Boot 启动类上一个 @SpringBootApplication 注解是
- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

三个注解组成的一个复合注解。其中 @SpringBootConfiguration 其实也是和@Configuration 注解组成的一个组合注解，功能也是和 @Configuration 相同；@ComponentScan 这个注解是标注需要扫描的包。今天着重说一下 @EnableAutoConfiguration 注解。
<!--more-->
---

   其实 @EnableAutoConfiguration 注解也和其它 @Enable\* 
注解一脉相乘的，简单说一下就是借助 @Import 的支持，收集和注册特定场景相关的Bean的定义：比如：@EnableAspectJAutoProxy 就是通过@Import 注解动态的将Bean注册到 SpringIoc 
容器中，而@EnableAutoConfiguration 是借助 @Import 把所有符合条件的 Bean 加载到 
SpringIoc 容器中。
> 参看上一篇文章 [Spring Boot 
自动配置之@Enable*与@Import注解](https://my.oschina.net/u/3555293/blog/3015439)

#### 1、@EnableAutoConfiguration 也是一个组合注解
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};
	
	String[] excludeName() default {};

}
```

其中最重要的是 @Import(AutoConfigurationImportSelector.class)注解。借助AutoConfigurationImportSelector ，@EnableAutoConfiguration 帮助Spring Boot 应用将所有符合条件的 @Configuration 配置加载到当前IoC容器中。而最主要的还是借助于 Spring 框架一的一个工具类：SpringFactoriesLoader 将 META-INF/spring.factories加载配置，spring.factories 文件是一个典型的properties配置文件，配置的格式仍然是Key = Value 的形式，中不过 Key 和 Value 都是Java的完整类名。比如：org.springframework.data.repository.core.support.RepositoryFactorySupport=org.springframework.data.jpa.repository.support.JpaRepositoryFactory

#### 2、AutoConfigurationImportSelector 源码
```java
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader); //1
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
				autoConfigurationMetadata, annotationMetadata); //2
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}

```
-  1.获取注解信息
-  2.获取所有配置列表

其中  AutoConfigurationMetadataLoader.loadMetadata方法源码

```java
public static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
		return loadMetadata(classLoader, PATH);
	}

	static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader, String path) {
		try {
			Enumeration<URL> urls = (classLoader != null) ? classLoader.getResources(path)
					: ClassLoader.getSystemResources(path);
			Properties properties = new Properties();
			while (urls.hasMoreElements()) {
				properties.putAll(PropertiesLoaderUtils
						.loadProperties(new UrlResource(urls.nextElement())));
			}
			return loadMetadata(properties);
		}
		catch (IOException ex) {
			throw new IllegalArgumentException(
					"Unable to load @ConditionalOnClass location [" + path + "]", ex);
		}
	}
```


getAutoConfigurationEntry 方法：

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(
			AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		//1.
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		//2.
		List<String> configurations = getCandidateConfigurations(annotationMetadata,
				attributes);
	    //3.
		configurations = removeDuplicates(configurations);
		//4.
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		//5.
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}

```

- 1.获取注解上所有属性信息
- 2.获取候选配置列表  [核心步骤]
    ```java
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
    ```
    通过SpringFactoriesLoader.loadFactoryNames获取配置信息
- 3.去重
- 4.配置exclude 信息去除不需要的
- 5.派发事件
