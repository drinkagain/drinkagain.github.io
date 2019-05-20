---
title: (05) SpringBoot 拦截器、过滤器、监听器
date: 2019-05-20 10:20:05
tags: SpringBoot
---
&emsp;&emsp;在工作中使用Web框架，总是避免不了与这些概念打交道，做一下总结，一口气说完拦截器、过滤器、监听器。

[GitHub源码地址](https://github.com/Zejun-Liu/SpringBoot/blob/master/springboot-interceptor-filter-listener/README.md)
### 1. 拦截器、过滤器、监听器区别
- 拦截器(interceptor)：依赖于web框架，基于Java的反射机制，属于AOP的一种应用。一个拦截器实例在一个controller生命周期内可以多次调用。只能拦截Controller的请求。
- 过滤器(Filter)：依赖于Servlet容器，基于函数回掉，可以对几乎所有请求过滤，一个过滤器实例只能在容器初使化调用一次。
- 监听器(Listener)：web监听器是Servlet中的特殊的类，用于监听web的特定事件，随web应用启动而启动，只初始化一次。
<!--more-->
### 2. 有什么用
- 拦截器(interceptor)：在一个请求进行中的时候，你想干预它的进展，甚至控制是否终止。这是拦截器做的事。
- 过滤器(Filter)：当有一堆东西，只希望选择符合的东西。定义这些要求的工具，就是过滤器。
- 监听器(Listener)：一个事件发生后，只希望获取这些事个事件发生的细节，而不去干预这个事件的执行过程，这就用到监听器

### 3. 启动顺序
    监听器 >  过滤器 > 拦截器
### 4.SpringBoot中的具体实现
#### （1） 拦截器
1. 拦截器常用有两种方式实现
    - 实现HandlerInterceptor接口
    - 继承HandlerInterceptorAdapter 抽象类
2. 区别和联系
 - HandlerInterceptorAdapter 实现AsyncHandlerInterceptor接口，AsyncHandlerInterceptor接口 继承HandlerInterceptor接口.
 - AsyncHandlerInterceptor接口多了一个afterConcurrentHandlingStarted方法
3. 具体方法
  - preHandle //请求过来之后首先走的方法 return true 继续往下执行
  - postHandle //请求之后返回之前
  - afterCompletion //处理完成之后
  - afterConcurrentHandlingStarted  //如果返回一个current类型的变量，会启用一个新的线程。执行完preHandle方法之后立即会调用afterConcurrentHandlingStarted,然后新线程再以次执行preHandle,postHandle,afterCompletion
4. 代码实现

> ***【注】以下代码基于springboot2.0***

(1)拦截器
    
MyInterceptor1  继承 HandlerInterceptorAdapter

MyInterceptor2  实现 HandlerInterceptor接口
```java 
public class MyInterceptor1 extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        request.setAttribute("startTime", System.currentTimeMillis());
        System.out.println(">>>>> MyInterceptor1 preHandle >>>>>>>>>>>>>>>>>>>>>>");
        return super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        long startTime = (long) request.getAttribute("startTime");
        System.out.println("MyInterceptor1 执行:" + (System.currentTimeMillis() - startTime));
        System.out.println(">>>>> MyInterceptor1 postHandle >>>>>>>>>>>>>>>>>>>>>>");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        request.removeAttribute("startTime");
        System.out.println(">>>>> MyInterceptor1 afterCompletion >>>>>>>>>>>>>>>>>>>>>>");
    }

    @Override
    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        super.afterConcurrentHandlingStarted(request, response, handler);
        System.out.println(">>>>> MyInterceptor1 afterConcurrentHandlingStarted >>>>>>>>>>>>>>>>>>>>>>");
    }
}
```
```java
public class MyInterceptor2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        request.setAttribute("startTime", System.currentTimeMillis());
        System.out.println(">>>>> MyInterceptor2 preHandle >>>>>>>>>>>>>>>>>>>>>>");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        long startTime = (long) request.getAttribute("startTime");
        System.out.println("MyInterceptor2 执行:" + (System.currentTimeMillis() - startTime));
        System.out.println(">>>>> MyInterceptor2 postHandle >>>>>>>>>>>>>>>>>>>>>>");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        request.removeAttribute("startTime");
        System.out.println(">>>>> MyInterceptor2 afterCompletion >>>>>>>>>>>>>>>>>>>>>>");
    }
}
```
 (2)配置
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor1()).addPathPatterns("/**");
        registry.addInterceptor(new MyInterceptor2()).addPathPatterns("/**");
    }
}
```
(3)请求
```java
@RestController
@SpringBootApplication
public class SpringbootInterceptorApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootInterceptorApplication.class, args);
    }


    @GetMapping(value = "/hello1")
    public ResponseEntity<String> hello() throws InterruptedException {
        Thread.sleep(500);
        return ResponseEntity.ok("HelloWorld");
    }

    @GetMapping(value = "/hello2")
    public StreamingResponseBody hello2() throws InterruptedException {
        Thread.sleep(500);
        return (OutputStream outputStream) -> {
            outputStream.write("success".getBytes());
            outputStream.flush();
            outputStream.close();
        };
    }

    @GetMapping(value = "/hello3")
    public Future<String> hello3() throws InterruptedException {
        Thread.sleep(500);
        return new AsyncResult<>("Hello");
    }
}
```
(4) 运行结果
1. 请求/hello1
    ```string
    >>>>> MyInterceptor1 preHandle >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor2 preHandle >>>>>>>>>>>>>>>>>>>>>>
    MyInterceptor2 执行:516
    >>>>> MyInterceptor2 postHandle >>>>>>>>>>>>>>>>>>>>>
    MyInterceptor1 执行:516
    >>>>> MyInterceptor1 postHandle >>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor2 afterCompletion >>>>>>>>>>>>>>>>
    >>>>> MyInterceptor1 afterCompletion >>>>>>>>>>>>>>>>
    ```
    > 执行按preHandle > postHandle > afterCompletion

2. 请求/hello2  或 /hello3
    ```string
    >>>>> MyInterceptor1 preHandle >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor2 preHandle >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor1 afterConcurrentHandlingStarted >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor1 preHandle >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor2 preHandle >>>>>>>>>>>>>>>>>>>>>>
    MyInterceptor2 执行:1
    >>>>> MyInterceptor2 postHandle >>>>>>>>>>>>>>>>>>>>>>
    MyInterceptor1 执行:1
    >>>>> MyInterceptor1 postHandle >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor2 afterCompletion >>>>>>>>>>>>>>>>>>>>>>
    >>>>> MyInterceptor1 afterCompletion >>>>>>>>>>>>>>>>>>>>>>
    ```
    > MyInterceptor1 执行顺序 preHandle > afterConcurrentHandlingStarted   > preHandle > postHandle >afterCompletion
    
    > MyInterceptor2 执行顺序 preHandle > preHandle > postHandle > afterCompletion
    
> 综上.对于concurrent类型的返回值，spring会启用一个新的线程来处理concurrent类型消息，在新的线程中会重新调用preHandle方法。
#### （2） 过滤器
(1) 过滤器

```java
public class MyFilter1 implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println(filterConfig.getInitParameter("initParam"));
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("doFilter1 >>>>>>>>>>>");
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```
(2) 配置

- 第一种方式
```java
@Bean
public FilterRegistrationBean<MyFilter1> filterRegistrationBean() {
    FilterRegistrationBean<MyFilter1> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.addUrlPatterns("/*");//过滤所有
    filterRegistrationBean.setFilter(new MyFilter1());
    filterRegistrationBean.setOrder(1);
    filterRegistrationBean.addInitParameter("initParam", "initOk");
    return filterRegistrationBean;
}
```
- 第二种方式

```java
@Bean
public MyFilter1 myFilter() {
    return new MyFilter1();
}
```
- 第三种方式
```java
@WebFilter("/test/*")
public class MyFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("MyFilter2");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("DoFilter 2");
    }
}
```
> 通过@WebFilter("/test/*")注解，首先需要@ServletComponentScan("com.jiuxian")

> Filter 全局拦截的配置(/*)和 Interceptor(/\*\*)有所区别需要注意

#### （3） 监听器
(1) 监听器
```java
public class MyListener1 implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("MyListener1 ... ");
    }
}
```
(2) 配置方式和Filter类似
-  第一种方式
```java
 @Bean
public ServletListenerRegistrationBean<MyListener1> registrationBean() {
    ServletListenerRegistrationBean<MyListener1> servletListenerRegistrationBean
            = new ServletListenerRegistrationBean<>();
    servletListenerRegistrationBean.setListener(new MyListener1());
    return servletListenerRegistrationBean;
}
```
- 第二种方式
```java
@Bean
public MyListener1 myListener1() {
    return new MyListener1();
}
```
- 第三种方式
```java
@WebListener
public class MyListener2 implements ServletRequestListener {

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("MyListener2");
    }
}
```
> 使用@WebListener注解，首先需要@ServletComponentScan("com.jiuxian")


> ***【注】以上代码基于springboot2.0***

[GitHub源码地址](https://github.com/Zejun-Liu/SpringBoot/blob/master/springboot-interceptor-filter-listener/README.md)