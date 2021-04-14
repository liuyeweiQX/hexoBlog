---
title: CompletableFuture异步任务
date: 2020-12-16 23:32:30
tags: 
  - 异步
categories: 
  - Java进阶
description: CompletableFuture异步任务
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img15.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img15.jpg
---
### 前言
所谓异步调用其实就是实现一个可无需等待被调用函数的返回值而让操作继续运行的方法。**在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果**。

#### Java实现多线程的三种方式
- 继承Thread类
run() 方法 和 start() 方法：
run() 方法：普通的成员方法
start() 方法：负责启动一个新的线程，并调用 run() 方法
**因此启动线程，需要使用 start() 方法**  
{% asset_img content1.png content1 %}   
  
- 实现 Runnable 接口
**实际上 Thread 类也是实现了 Runnable 接口：**
class Thread implements Runnable {
**启动 Runnable 实例时，需要放在 Thread 中，然后调用 start() 方法**     
{% asset_img content2.png content2 %}  

- 实现 Callable 接口
Java 5 开始提供
可以返回结果（通过 Future），也可以抛出异常
需要实现的是 call() 方法
**以上两点也是 Callable 接口 与 Runnable 接口的区别**
{% asset_img content3.png content3 %}     

#### Java FutureTask可异步执行的任务
FutureTask 是 Future 接口的一个实现类。
使用方式：
- 构造一个 Callable 接口的实例
- 构造一个 FutureTask 的实例，封装 Callable 接口的实例
- 通过线程池 ExecutorService 来运行 FutureTask 的实例
- 通过 get() 方法来获得结果
**可以看出，FutureTask 与使用 Callable & Future 的目的差不多，因此 FutureTask 实际不常使用，直接使用 Callable & Future 就可以了**。   
{% asset_img content4.png content4 %}  

JDK5 新增了 Future 接口，用于描述一个异步计算的结果。虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，**只能通过阻塞或者轮询的方式得到任务的结果**。 例如：
{% asset_img content5.png content5 %}

阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果，**为什么不能用观察者设计模式呢？即当计算结果完成及时通知监听者。（例如通过回调的方式）** 
      
### CompletableFuture
Java 8 中, 新增加了一个包含 50 个方法左右的类 CompletableFuture，它提供了非常强大的 Future 的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法。
 
对于阻塞或者轮询方式，依然可以通过 CompletableFuture 类的 CompletionStage 和 Future 接口方式支持。
 
CompletableFuture 类声明了 CompletionStage 接口，CompletionStage 接口实际上提供了同步或异步运行计算的舞台，所以我们可以通过实现多个 CompletionStage 命令，并且将这些命令串联在一起的方式实现多个命令之间的触发。
 
我们可以通过 CompletableFuture.supplyAsync(this::sendMsg); 这么一行代码创建一个简单的异步计算。在这行代码中，supplyAsync 支持异步地执行我们指定的方法，这个例子中的异步执行方法是 sendMsg。当然，我们也可以使用 Executor 执行异步程序，默认是 ForkJoinPool.commonPool()。
我们也可以在异步计算结束之后指定回调函数，例如 CompletableFuture.supplyAsync(this::sendMsg) .thenAccept(this::notify); 
这行代码中的 thenAccept 被用于增加回调函数，在我们的示例中 notify 就成了异步计算的消费者，它会处理计算结果。   

### CompletionStage<T> 接口
一个可能执行的异步计算的某个阶段，在另一个CompletionStage完成时执行一个操作或计算一个值。
一个阶段完成后，其计算结束。但是，该计算阶段可能会触发下一个计算阶段。
最简单的例子
CompletableFuture 实际上也实现了 Future 接口： 
{% asset_img content6.png content6 %}      

所以我们也可以利用 CompletableFuture 来实现基本的 Future 功能，例如：
{% asset_img content7.png content7 %} 

此时此刻主线程 future.get() 将得到字符串的结果 I have completed，同时完成回调以后将会立即生效。注意 complete() 方法只能调用一次，后续调用将被忽略。
注意：get() 方法可能会抛出异常 InterruptedException 和 ExecutionException。
如果我们已经知道了异步任务的结果，我们也可以直接创建一个已完成的 future，如下：
{% asset_img content8.png content8 %} 

如果在异步执行过程中，我们觉得执行会超时或者会出现问题，我们也可以通过 cancle() 方法取消，此时调用 get() 方法时会产生异常 java.util.concurrent.CancellationException，代码如下：
{% asset_img content9.png content9 %} 

### 使用工厂方法创建 CompletableFuture
在上述的代码中，我们手动地创建 CompletableFuture，并且手动的创建一个线程（或者利用线程池）来启动异步任务，这样似乎有些复杂。
 
其实我们可以利用 CompletableFuture 的工厂方法，传入 Supplier 或者 Runnable 的实现类，直接得到一个 CompletableFuture 的实例：
```
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
    public static CompletableFuture<Void> runAsync(Runnable runnable)
    public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
```
第一个和第三个方法，没有 Executor 参数，将会使用 ForkJoinPool.commonPool() (全局的，在 JDK8 中介绍的通用池），这适用于 CompletableFuture 类中的大多数的方法。
- Runnable 接口方法 public abstract void run(); 没有返回值
- Supplier 接口方法 T get(); 有返回值。**如果你需要处理异步操作并返回结果，使用前两种 Supplier<U> 方法**
使用 Lambda 表达式来传入 Supplier 或者 Runnable 的实现类。 
{% asset_img content10.png content10 %}  

### 转换和作用于异步任务的结果 (thenApply)
我们可以叠加功能，把多个 future 组合在一起等
``` 
    public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
```
该方法的作用是在该计算阶段正常完成后，将该计算阶段的结果作为参数传递给参数 fn 值的函数Function，并会返回一个新的 CompletionStage
```
    public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
```
该方法和上面的方法 thenApply 功能类似，不同的是对该计算阶段的结果进行计算的函数 fn 的执行时异步的。
```
    public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```
该方法和上面的方法 thenApplyAsync 功能类似，不同的是对该计算阶段的结果进行计算的函数 fn 的执行时异步的， 并且是在调用者提供的线程池中执行的。
Function 接口方法 R apply(T t); 包含一个参数和一个返回值
一个示例代码如下：
{% asset_img content11.png content11 %} 

### 运行完成的异步任务的结果 (thenAccept/thenRun)
在 future 的管道里有两种典型的“最终”阶段方法。他们在你使用 future 的值的时候做好准备，当 thenAccept() 提供最终的值时，thenRun 执行 Runnable。
```
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)
    public CompletableFuture<Void> thenRun(Runnable action)
    public CompletableFuture<Void> thenRunAsync(Runnable action)
    public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)
```
Consumer 接口方法 void accept(T t); 包含一个参数，但是没有返回值
一个示例代码如下：
{% asset_img content12.png content12 %} 

### 结合两个 CompletableFuture
```
    public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)
    //传入前一个 CompletableFuture 的返回值，返回另外一个 CompletableFuture 实例
    public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn)
    public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor)
    public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
    public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
    public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)
```
一个示例代码如下：
{% asset_img content13.png content13 %} 

上述功能也可以通过 thenCombine() 方法实现，传入一个 BiFunction 接口的实例（以 Lambda 形式） 例如： 
{% asset_img content14.png content14 %} 

### 并行执行多个异步任务
有时候我们可能需要等待所有的异步任务都执行完毕，然后组合他们的结果。我们可以使用 allOf() 方法：
```
   public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
```
一个示例代码如下：
{% asset_img content15.png content15 %}

有时候我们可能**不需要**等待所有的异步任务都执行完毕，只要任何一个任务完成就返回结果。我们可以使用 anyOf() 方法：
```
   public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```
一个示例代码如下：
{% asset_img content16.png content16 %}
         
### 异常的处理
我们可以在 handle() 方法里处理异常：
```
   public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn) 
```
第一个参数为 CompletableFuture 返回的结果
第二个参数为抛出的异常
一个示例代码如下：  
{% asset_img content17.png content17 %}                           