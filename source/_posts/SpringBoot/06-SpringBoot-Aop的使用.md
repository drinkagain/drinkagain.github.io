---
title: (06) SpringBoot Aop的使用
date: 2019-05-20 10:20:53
tags: SpringBoot
---
AOP：面向切面编程，相对于OOP面向对角编程
Spring的AOP的存在目的是为了解耦。AOP可以让一组类共享相同的行为。在OOP中只能继承和实现接口，且类继承只能单继承，阻碍更多行为添加到一组类上，AOP弥补了OOP的不足。

还有就是为了清晰的逻辑，让业务逻辑关注业务本身，不用去关心其它的事情，比如事务。

Spring的AOP是通过JDK的动态代理和CGLIB实现的。

### 一、AOP的术语：
<!--more-->w
aop 有一堆术语，非常难以理解，简单说一下
- 通知(有的地方叫增强)(Advice) 

  需要完成的工作叫做通知，就是你写的业务逻辑中需要比如事务、日志等先定义好，然后需要的地方再去用
- 连接点(Join point) 

  就是spring中允许使用通知的地方，基本上每个方法前后抛异常时都可以是连接点
- 切点(Poincut) 

  其实就是筛选出的连接点，一个类中的所有方法都是连接点，但又不全需要，会筛选出某些作为连接点做为切点。如果说通知定义了切面的动作或者执行时机的话，切点则定义了执行的地点
- 切面(Aspect) 

  其实就是通知和切点的结合，通知和切点共同定义了切面的全部内容，它是干什么的，什么时候在哪执行
- 引入(Introduction) 

  在不改变一个现有类代码的情况下，为该类添加属性和方法,可以在无需修改现有类的前提下，让它们具有新的行为和状态。其实就是把切面（也就是新方法属性：通知定义的）用到目标类中去
- 目标(target) 

  被通知的对象。也就是需要加入额外代码的对象，也就是真正的业务逻辑被组织织入切面。
- 织入(Weaving) 

  把切面加入程序代码的过程。切面在指定的连接点被织入到目标对象中，在目标对象的生命周期里有多个点可以进行织入：
  + 编译期：切面在目标类编译时被织入，这种方式需要特殊的编译器
  + 类加载期：切面在目标类加载到JVM时被织入，这种方式需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码
  + 运行期：切面在应用运行的某个时刻被织入，一般情况下，在织入切面时，AOP容器会为目标对象动态创建一个代理对象，Spring AOP就是以这种方式织入切面的。

**例：**
```java
public class UserService{
    void save(){}
    List list(){}
    ....
}
```
在UserService中的save()方法前需要开启事务，在方法后关闭事务，在抛异常时回滚事务。

那么,UserService中的所有方法都是连接点(JoinPoint),save()方法就是切点(Poincut)。需要在save()方法前后执行的方法就是通知(Advice)，切点和通知合起来就是一个切面(Aspect)。save()方法就是目标(target)。把想要执行的代码动态的加入到save()方法前后就是织入(Weaving)。

有的地方把通知称作增强是有道理的，在业务方法前后加上其它方法，其实就是对该方法的增强。

### 二、常用AOP通知(增强)类型

- before(前置通知)：  在方法开始执行前执行
- after(后置通知)：  在方法执行后执行
- afterReturning(返回后通知)：   在方法返回后执行
- afterThrowing(异常通知)： 在抛出异常时执行
- around(环绕通知)：  在方法执行前和执行后都会执行

### 三、执行顺序
> around > before > around > after > afterReturning

### 四、先说一下SpringAop非常霸道又用的非常少的功能 --引入(Introduction) 
1. 配置类
```java
@Aspect
@Component
public class IntroductionAop {

    @DeclareParents(value = "com.jiuxian..service..*", defaultImpl = DoSthServiceImpl.class)
    public DoSthService doSthService;

}
```
2. service代码
```java
public interface DoSthService {

    void doSth();
}

@Service
public class DoSthServiceImpl implements DoSthService {

    @Override
    public void doSth() {
        System.out.println("do sth ....");
    }
    
}

public interface UserService {

    void testIntroduction();
}

@Service
public class UserServiceImpl implements UserService {

    @Override
    public void testIntroduction() {
        System.out.println("do testIntroduction");
    }
}
```
3. 测试代码
```java
@Test
public void testIntroduction() {
    userService.testIntroduction();
    //Aop 让UserService方法拥有 DoSthService的方法
    DoSthService doSthService = (DoSthService) userService;
    doSthService.doSth();
}
```
4. 结果
```string
do testIntroduction
do sth ....
```

### 五、五种通知（增强）代码实现
1. 配置类

(1) 对方法
```java
@Aspect
@Component
public class TransactionAop {

    @Pointcut("execution(* com.jiuxian..service.*.*(..))")
    public void pointcut() {
    }

    @Before("pointcut()")
    public void beginTransaction() {
        System.out.println("before beginTransaction");
    }

    @After("pointcut()")
    public void commit() {
        System.out.println("after commit");
    }

    @AfterReturning("pointcut()", returning = "returnObject")
    public void afterReturning(JoinPoint joinPoint, Object returnObject) {
        System.out.println("afterReturning");
    }

    @AfterThrowing("pointcut()")
    public void afterThrowing() {
        System.out.println("afterThrowing afterThrowing  rollback");
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            System.out.println("around");
            return joinPoint.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
            throw e;
        } finally {
            System.out.println("around");
        }
    }
}
```
(2) 对注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Log {
    String value() default "";
}
```

```java
@Aspect
@Component
public class AnnotationAop {

    @Pointcut(value = "@annotation(log)", argNames = "log")
    public void pointcut(Log log) {
    }

    @Around(value = "pointcut(log)", argNames = "joinPoint,log")
    public Object around(ProceedingJoinPoint joinPoint, Log log) throws Throwable {
        try {
            System.out.println(log.value());
            System.out.println("around");
            return joinPoint.proceed();
        } catch (Throwable throwable) {
            throw throwable;
        } finally {
            System.out.println("around");
        }
    }
}

    @Before("@annotation(com.jiuxian.annotation.Log)")
    public void before(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Log log = method.getAnnotation(Log.class);
        System.out.println("注解式拦截 " + log.value());
    }
```
2. service 方法实现

```java
public interface UserService {

    String save(String user);

    void testAnnotationAop();
}


@Service
public class UserServiceImpl implements UserService {

    @Override
    public String save(String user) {
        System.out.println("保存用户信息");
        if ("a".equals(user)) {
            throw new RuntimeException();
        }
        return user;
    }

    @Log(value = "test")
    @Override
    public void testAnnotationAop() {
        System.out.println("testAnnotationAop");
    }
}
```
3. 测试类
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootAopApplicationTests {

    @Resource
    private UserService userService;

    @Test
    public void testAop1() {
        userService.save("张三");
        Assert.assertTrue(true);
    }

    @Test
    public void testAop2() {
        userService.save("a");
    }
    
    @Test
    public void testAop3() {
        userService.testAnnotationAop();
    }
}
```
4. 结果
- 执行testAop1时
```string
around
before beginTransaction
保存用户信息
around
after commit
afterReturning :: 张三
```
- 执行testAop2时
```string
around
before beginTransaction
保存用户信息
around
after commit
afterThrowing  rollback
```
- 执行testAop3时
```string
test
around
testAnnotationAop
around
```
5. pom文件
```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```
### 六、最常用的execution解释
> 例: execution(\* com.jiuxian..service.\*.\*(..))

- execution 表达式的主体
- 第一个* 代表任意的返回值
- com.jiuxian aop所横切的包名
- 包后面.. 表示当前包及其子包
- 第二个* 表示类名，代表所有类
- .*(..) 表示任何方法,括号代表参数 .. 表示任意参数 

> 例: execution(\* com.jiuxian..service.\*Service.add\*(String))

表示： com.jiuxian 包及其子包下的service包下，类名以Service结尾，方法以add开头，参数类型为String的方法的切点。

### 七、特别的用法
```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```
可以使用 &&, ||, ! 运算符来定义切点

### 八、更多详细介绍请参阅官网
[SpringAOP官网介绍](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts)


### 九、本文示例代码

[GitHub 源码](https://github.com/Zejun-Liu/SpringBoot/blob/master/springboot-aop/README.md)


> 以上代码基于Springboot 2.0

