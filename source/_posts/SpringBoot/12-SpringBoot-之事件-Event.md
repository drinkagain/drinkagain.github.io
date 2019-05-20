---
title: (12) SpringBoot 之事件(Event)
date: 2019-05-20 10:26:59
tags: SpringBoot
---
Spring 官方文档翻译如下 ：

*ApplicationContext 通过 ApplicationEvent 类和 ApplicationListener 接口进行事件处理。 如果将实现 ApplicationListener 接口的 bean 注入到上下文中，则每次使用 ApplicationContext 发布 ApplicationEvent 时，都会通知该 bean。 本质上，这是标准的观察者设计模式。*

Spring的事件（Application Event）其实就是一个观察者设计模式，一个 Bean 处理完成任务后希望通知其它 Bean 或者说 一个Bean 想观察监听另一个Bean的行为。
<!--more-->
Spring 事件只需要几步：
- 自定义事件，继承 ApplicationEvent
- 定义监听器，实现 ApplicationListener 或者通过 @EventListener 注解到方法上
- 定义发布者，通过 ApplicationEventPublisher 

代码示例：

#### 1. 自定义Event

```java
@Data
public class DemoEvent extends ApplicationEvent {
    private Long id;
    private String message;

    public DemoEvent(Object source, Long id, String message) {
        super(source);
        this.id = id;
        this.message = message;
    }
}
```

#### 2. 监听器
- 实现ApplicationListener 接口
```java

@Component
public class DemoListener implements ApplicationListener<DemoEvent> {

    @Override
    public void onApplicationEvent(DemoEvent demoEvent) {
        System.out.println(">>>>>>>>>DemoListener>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        System.out.println("收到了：" + demoEvent.getSource() + "消息;时间：" + demoEvent.getTimestamp());
        System.out.println("消息：" + demoEvent.getId() + ":" + demoEvent.getMessage());
    }
}
```
> 泛型为需要监听的事件类型

- @EventListener
```java
@Component
public class DemoListener2 {

    @EventListener
    public void onApplicationEvent(DemoEvent demoEvent) {
        System.out.println(">>>>>>>>>DemoListener2>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        System.out.println("收到了：" + demoEvent.getSource() + "消息;时间：" + demoEvent.getTimestamp());
        System.out.println("消息：" + demoEvent.getId() + ":" + demoEvent.getMessage());
    }
}

```

> 参数为需要监听的事件类型

#### 3. 消息发布者

```java
@Component
public class DemoPublisher {

    private final ApplicationContext applicationContext;

    @Autowired
    public DemoPublisher(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    public void publish(long id, String message) {
        applicationContext.publishEvent(new DemoEvent(this, id, message));
    }

}
```

#### 4. 测试方法

```java
@Test
public void publisherTest() {
    demoPublisher.publish(1L, "成功了！");
}
```

#### 5.结果
```test
>>>>>>>>>DemoListener2>>>>>>>>>>>>>>>>>>>>>>>>>>>>
收到了：com.jiuxian.publisher.DemoPublisher@3a62c01e消息;时间：1551762322376
消息：1:成功了！
>>>>>>>>>DemoListener>>>>>>>>>>>>>>>>>>>>>>>>>>>>
收到了：com.jiuxian.publisher.DemoPublisher@3a62c01e消息;时间：1551762322376
消息：1:成功了！

```

#### 6. 示例源码
[GitHub  https://github.com/Zejun-Liu/SpringBoot2.0/tree/master/springboot-event](https://github.com/Zejun-Liu/SpringBoot2.0/tree/master/springboot-event)

