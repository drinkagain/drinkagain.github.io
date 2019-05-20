---
title: (02) Future模式之CompletableFuture
date: 2019-05-20 10:29:49
tags: Java
---
CompletableFuture 是Java 8 新增加的Api,该类实现，Future和CompletionStage两个接口，提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，提供了函数式编程的能力，可以通过回调的方式处理计算结果，并且提供了转换和组合CompletableFuture的方法。

### 一、主动完成计算
<!--more-->
- public T get() 

    该方法为阻塞方法，会等待计算结果完成
    
- public T get(long timeout,TimeUnit unit)
    
    有时间限制的阻塞方法

- public T getNow(T valueIfAbsent)

    立即获取方法结果，如果没有计算结束则返回传的值
    
- public T join()

    和 get() 方法类似也是主动阻塞线程，等待计算结果。和get() 方法有细微的差别

```java
public class ThreadUtil {
    public static void sleep(long ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
            throw new RuntimeException(e.getMessage());
        }
    }
}

```
    
```java
public static void main(String[] args) {

     CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            ThreadUtil.sleep(200);
            return 10 / 0;
    });
//       System.out.println(future.join());
//       System.out.println(future.get());
    System.out.println(future.getNow(10));
}
```
    
- public boolean complete(T value) 

    立即完成计算，并把结果设置为传的值，返回是否设置成功
    
    如果 CompletableFuture 没有关联任何的Callback、异步任务等，如果调用get方法，那会一直阻塞下去，可以使用complete方法主动完成计算
    
```java
public static void main(String[] args) throws Exception {
    CompletableFuture<Integer> future = new CompletableFuture<>();
//        future.get();
    future.complete(10);
}
```
- public boolean completeExceptionally(Throwable ex)
    立即完成计算，并抛出异常

### 二、执行异步任务

创建一个异步任务
    
- public static \<U\> CompletableFuture\<U\>    completedFuture(U value)
 
    创建一个有初始值的CompletableFuture

- public static CompletableFuture<Void> 	runAsync(Runnable runnable)
- public static CompletableFuture<Void> 	runAsync(Runnable runnable, Executor executor)
- public static \<U> CompletableFuture\<U> 	supplyAsync(Supplier\<U> supplier)
- public static \<U> CompletableFuture\<U> 	supplyAsync(Supplier\<U> supplier, Executor executor)

    以上四个方法中，以 Async 结尾并且没有 Executor 参数的，会默认使用 ForkJoinPool.commonPool() 作为它的线程池执行异步代码。
    以run开头的，因为以 Runable 类型为参数所以没有返回值。示例：
```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> System.out.println("runAsync"));
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "supplyAsync");
        
        System.out.println(future1.get());
        System.out.println(future2.get());
    }
```
结果：
```string
runAsync
null
supplyAsync
```

### 三、计算完成时对结果的处理 whenComplete/exceptionally/handle
当CompletableFuture的计算结果完成，或者抛出异常的时候，我们可以执行特定的Action。主要是下面的方法：
- public CompletableFuture<T> 	whenComplete(BiConsumer<? super T,? super Throwable> action)
- public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
- public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)

参数类型为 BiConsumer<? super T, ? super Throwable> 会获取上一步计算的计算结果和异常信息。以Async结尾的方法可能会使用其它的线程去执行,如果使用相同的线程池，也可能会被同一个线程选中执行,以下皆相同。

```java
   public static void main(String[] args) throws Exception {
    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        ThreadUtil.sleep(100);
        return 20;
    }).whenCompleteAsync((v, e) -> {
        System.out.println(v);
        System.out.println(e);
    });
    System.out.println(future.get());
}
```
- public CompletableFuture<T>  exceptionally(Function<Throwable,? extends T> fn)

    该方法是对异常情况的处理，当函数异常时应该的返回值
    
```java
    public static void main(String[] args) throws Exception {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            ThreadUtil.sleep(100);
            return 10 / 0;
        }).whenCompleteAsync((v, e) -> {
            System.out.println(v);
            System.out.println(e);
        }).exceptionally((e) -> {
            System.out.println(e.getMessage());
            return 30;
        });
        System.out.println(future.get());
    }
```
---

- public \<U\>CompletableFuture\<U\> handle(BiFunction<? super T,Throwable,? extends U> fn)
- public \<U\>CompletableFuture\<U\> handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
- public \<U\> CompletableFuture\<U\> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)

handle 方法和whenComplete方法类似，只不过接收的是一个 BiFunction<? super T,Throwable,? extends U> fn 类型的参数，因此有 whenComplete 方法和 转换的功能 (thenApply)

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture
                .supplyAsync(() -> 10 / 0)
                .handle((t, e) -> {
                    System.out.println(e.getMessage());
                    return 10;
                });

        System.out.println(future.get());
    }
```

### 四、结果处理转换 thenApply

 CompletableFuture 由于有回调，可以不必等待一个计算完成而阻塞着调用县城，可以在一个结果计算完成之后紧接着执行某个Action。我们可以将这些操作串联起来。

- public \<U\> CompletableFuture\<U\> 	thenApply(Function<? super T,? extends U> fn)
- public \<U\> CompletableFuture\<U\> 	thenApplyAsync(Function<? super T,? extends U> fn)
- public \<U\> CompletableFuture\<U\> 	thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)

```java
 public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture
                .supplyAsync(() -> 1)
                .thenApply((a) -> {
                    System.out.println(a);//1
                    return a * 10;
                }).thenApply((a) -> {
                    System.out.println(a);//10
                    return a + 10;
                }).thenApply((a) -> {
                    System.out.println(a);//20
                    return a - 5;
                });
        System.out.println(future.get());//15
    }
```

这些方法不是马上执行的，也不会阻塞，而是前一个执行完成后继续执行下一个。

和 handle 方法的区别是，handle 会处理正常计算值和异常，不会抛出异常。而 thenApply 只会处理正常计算值，有异常则抛出。

### 五、纯消费 thenAccept/thenAcceptBoth/thenRun
  单纯的去消费结果而不会返回新的值，因些计算结果为 Void;

- public CompletableFuture<Void> 	thenAccept(Consumer<? super T> action)
- public CompletableFuture<Void> 	thenAcceptAsync(Consumer<? super T> action)
- public CompletableFuture<Void> 	thenAcceptAsync(Consumer<? super T> action, Executor executor)

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        
        CompletableFuture<Void> future = CompletableFuture
                .supplyAsync(() -> 1)
                .thenAccept(System.out::println) //消费 上一级返回值 1
                .thenAcceptAsync(System.out::println); //上一级没有返回值 输出null
                
        System.out.println(future.get()); //消费函数没有返回值 输出null
    }
```
--- 
- public \<U\> CompletableFuture<Void> 	thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
- public \<U\> CompletableFuture<Void> 	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
- public \<U\> CompletableFuture<Void> 	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)

和 thenAccept 相比，参数类型多了一个 CompletionStage<? extends U> other,以上方法会接收上一个CompletionStage返回值，和当前的一个。

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture
                .supplyAsync(() -> 1)
                .thenAcceptBoth(CompletableFuture.supplyAsync(() -> 2), (a, b) -> {
                    System.out.println(a);
                    System.out.println(b);
                }).get();
    }
```

- public     CompletableFuture<Void> 	runAfterBoth(CompletionStage<?> other,  Runnable action)

  runAfterBoth 和以上方法不同，传一个 Runnable 类型的参数，不接收上一级的返回值
  
---
 更彻底的：

- public CompletableFuture<Void>	thenRun(Runnable action)
- public CompletableFuture<Void>	thenRunAsync(Runnable action)
- public CompletableFuture<Void> 	thenRunAsync(Runnable action, Executor executor)

以上是彻底的纯消费，完全忽略计算结果

### 六、组合 thenCompose/thenCombine

- public \<U\> CompletableFuture\<U\> 	thenCompose(Function<? super T,? extends CompletionStage\<U>> fn)
- public \<U\> CompletableFuture\<U\> 	thenComposeAsync(Function<? super T,? extends CompletionStage\<U>> fn)
- public \<U\> CompletableFuture\<U\> 	thenComposeAsync(Function<? super T,? extends CompletionStage\<U>> fn, Executor executor)

以上接收类型为 Function<? super T,? extends CompletionStage\<U\>> fn ,fn 接收上一级返回的结果，并返回一个新的 CompletableFuture

```java
   public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<Integer> future = CompletableFuture
                .supplyAsync(() -> 1)
                .thenApply((a) -> {
                    ThreadUtil.sleep(1000);
                    return a + 10;
                })
                .thenCompose((s) -> {
                    System.out.println(s); //11
                    return CompletableFuture.supplyAsync(() -> s * 5);
                });

        System.out.println(future.get());//55
    }
```
---

- public <U,V> CompletableFuture<V> 	thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
- public <U,V> CompletableFuture<V> 	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
- public <U,V> CompletableFuture<V> 	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)

两个CompletionStage是并行执行的，它们之间并没有先后依赖顺序，other并不会等待先前的CompletableFuture执行完毕后再执行。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

        Random random = new Random();
        CompletableFuture<Integer> future = CompletableFuture
                .supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    System.out.println("supplyAsync");
                    return 2;
                }).thenApply((a) -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    System.out.println("thenApply");
                    return a * 3;
                })
                .thenCombine(CompletableFuture.supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    System.out.println("thenCombineAsync");
                    return 10;
                }), (a, b) -> {
                    System.out.println(a);
                    System.out.println(b);
                    return a + b;
                });

        System.out.println(future.get());
    }
```
thenCombine 和 supplyAsync 不一定哪个先哪个后，是并行执行的。

### 七、acceptEither / applyToEither
- public CompletableFuture<Void> 	acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
- public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
- public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)

先看示例：
```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Random random = new Random();
        CompletableFuture
                .supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    return "A";
                })
                .acceptEither(CompletableFuture.supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    return "B";
                }), System.out::println)
                .get();
    }
```
以上代码有时输出A，有时输出B，哪个Future先执行完就会根据它的结果计算。
 
acceptEither方法是当任意一个 CompletionStage 完成的时候，action 这个消费者就会被执行。这个方法返回 CompletableFuture<Void>

---

- public \<U> CompletableFuture\<U> 	applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)
- public \<U> CompletableFuture\<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)
- public \<U> CompletableFuture\<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)


applyToEither 方法是当任意一个 CompletionStage 完成的时候，fn会被执行，它的返回值会当作新的CompletableFuture\<U>的计算结果。


> acceptEither 没有返回值，applyToEither 有返回值
### 八、allOf / anyOf

- public static CompletableFuture<Void> 	    allOf(CompletableFuture<?>... cfs)

这个方法的意思是把有方法都执行完才往下执行，没有返回值

```java
   public static void main(String[] args) throws ExecutionException, InterruptedException {

        Random random = new Random();
        CompletableFuture.allOf(
                CompletableFuture.runAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    System.out.println(1);
                }),
                CompletableFuture.runAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    System.out.println(2);
                }))
                .get();

    }
```
有时输出1 2 有时输出 2 1

--- 

- public static CompletableFuture<Object> 	anyOf(CompletableFuture<?>... cfs)

任务一个方法执行完都往下执行，返回一个Object类型的值

```java
  public static void main(String[] args) throws ExecutionException, InterruptedException {
        Random random = new Random();

        Object obj = CompletableFuture.anyOf(
                CompletableFuture.supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    return 1;
                }),
                CompletableFuture.supplyAsync(() -> {
                    ThreadUtil.sleep(random.nextInt(1000));
                    return 2;
                })).get();

        System.out.println(obj);
    }
```

输出结果有时为1 有时间为 2 