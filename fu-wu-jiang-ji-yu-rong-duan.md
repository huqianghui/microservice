概述：

微服务中对服务的限流，降级，熔断的需求是多种多样的，需要在API网关和各个具体服务接口中分别进行控制，才能满足复杂场景下微服务架构的应用需求。比如在虚拟机里面，不通过kubernetnes和docker对操作系统级别（内存，CPU，网络等）的隔离的话，一些极端情况某些服务也会导致整个虚拟机的服务全部死掉。再者熔断是在微服务之间的调用的一种应用的保证机制，但是API网管不做限流当外面请求海量进来的时候还是会把各个服务击破。限流最在网关章节说道，这里主要讲其它都好的情况下，熔断和降级的需求吧。

名词解释：  
**雪崩效应**

在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者”的不可用导致“服务消费者”的不可用,并将不可用逐渐放大的过程。

如果下图所示：A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。很常见发生的情况就是当某个服务锁定了一些资源的时候，包括数据库连接和java的锁都会导致长时间的等待，资源没有办法释放。

![](/assets/snowslide.png)

**熔断器（CircuitBreaker）**

熔断器的原理很简单，如同电力过载保护器。它可以实现快速失败，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。熔断器模式就像是那些容易导致错误的操作的一种代理。这种代理能够记录最近调用发生错误的次数，然后决定使用允许操作继续，或者立即返回错误。  
熔断器开关相互转换的逻辑如下图：

![](/assets/circuitBreaker.png)

**降级 \(Fallback\)**  
 当服务调用失败的时候，包括http的4XX，5XX错误都会调用的备选服务多为本地服务，除配置的忽略的异常。当断路器打开的时候就直接调用降级服务。如果没有配置降级服务，就把异常直接抛出。

**资源隔离两种策略：**

Hystrix就是用来做资源隔离的，比如说，当客户端向服务端发送请求时，给服务A分配了10个线程或者信号量，只要超过了这个并发量就走降级服务，就算服务A挂了，最多也就导致服务A不可用，容器的10个线程不可用了，但是不会影响系统中的其他服务。同时当资源有限的时候可以把资源分给更加重要的服务。

Hystrix的资源隔离策略有两种，分别为：线程池和信号量。

线程池：

容器分配给服务一个线程池，一定数量的可用线程数。当里面的线程执行完任务之后，就会将调用的结果返回给容器的线程，从而实现资源的隔离，当有大量并发的时候，服务内部的线程池的数量就决定了整个服务的并发度。

信号量：

信号量的资源隔离只是起到一个开关的作用，例如，服务X的信号量大小为10，那么同时只允许10个web容器的线程来访问服务X，其他的请求就会被拒绝，从而达到限流保护的作用。

二者的比较：

|  | **线程池隔离** | **信号量隔离** |
| :--- | :--- | :--- |
| 线程 | 与调用线程非相同线程 | 与调用线程相同（jetty线程） |
| 开销 | 排队、调度、上下文开销等 | 无线程切换，开销低 |
| 异步 | 支持 | 不支持 |
| 并发支持 | 支持（最大线程池大小） | 支持（最大信号量上限） |

从功能上来看线程池有更多的功能，但是开销要大一些。当请求是毫秒级别的时候建议用信号量，而当请求都是百毫秒级别的时候，线程的开销就可以忽略。这种情况建议使用线程隔离方案。

**配置参数说明**

上面的已经说了hystrix资源隔离的一个配置参数。接下来都hystrix的参数列举和说明一下，因为熔断相对还陌生，具体的代码可以参考HystrixCommandProperties这个类。

**Execution相关的属性的配置：**

hystrix.command.default.execution.isolation.strategy 隔离策略：

```
     默认是Thread, 可选Thread｜Semaphore

     thread 通过线程数量来限制并发请求数，可以提供额外的保护，但有一定的延迟。一般用于网络调用

      semaphore 通过semaphore count来限制并发请求数，适用于低延迟的网络的高并发请求
```

hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds 命令执行超时时间：

```
     默认1000ms
```

hystrix.command.default.execution.timeout.enabled 执行是否启用超时：

```
      默认启用true
```

hystrix.command.default.execution.isolation.thread.interruptOnTimeout 发生超时是是否中断：

```
       默认true
```

hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests 最大并发请求数：

```
          默认10，该参数当使用ExecutionIsolationStrategy.SEMAPHORE策略时才有效。如果达到最大并发请求数，请求会被拒绝。

         理论上选择semaphore size的原则和选择thread size一致，但选用semaphore时每次执行的单元要比较小且执行速度快（ms级别），否则的话应该用thread。semaphore应该占整个容器（tomcat）的线程池的一小部分。
```

**Fallback相关的属性**

这些参数可以应用于Hystrix的THREAD和SEMAPHORE策略。

hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests ：

```
        如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用。默认10
```

hystrix.command.default.fallback.enabled：

```
     当执行失败或者请求被拒绝，是否会尝试调用hystrixCommand.getFallback\(\) 。默认true
```

**Circuit Breaker相关的属性**

hystrix.command.default.circuitBreaker.enabled 用来跟踪circuit的健康性:

```
   如果未达标则让request短路。默认true
```

hystrix.command.default.circuitBreaker.requestVolumeThreshold 一个rolling window内最小的请求数:

```
    如果设为20，那么当一个rolling window的时间内（比如说1个rolling window是10秒）收到19个请求，即使19个请求都失败，也不会触发circuit break。默认20
```

hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds 触发短路的时间值:

```
  当该值设为5000时，则当触发circuit break后的5000毫秒内都会拒绝request，也就是5000毫秒后才会关闭circuit。默认5000
```

hystrix.command.default.circuitBreaker.errorThresholdPercentage错误比率阀值:

```
  如果错误率&gt;=该值，circuit会被打开，并短路所有请求触发fallback。默认50
```

hystrix.command.default.circuitBreaker.forceOpen 强制打开熔断器:

```
 如果打开这个开关，那么拒绝所有request，默认false
```

hystrix.command.default.circuitBreaker.forceClosed 强制关闭熔断器如果这个开关打开:

```
   circuit将一直关闭且忽略circuitBreaker.errorThresholdPercentage
```

**Metrics相关参数**

hystrix.command.default.metrics.rollingStats.timeInMilliseconds 设置统计的时间窗口值的，毫秒值:

```
          circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，则rolling window会被分                   

         成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
```

hystrix.command.default.metrics.rollingStats.numBuckets 设置一个rolling window被划分的数量:

```
           若numBuckets＝10，rolling window＝10000，那么一个bucket的时间即1秒。必须符合rolling window % numberBuckets       

            == 0。默认10
```

hystrix.command.default.metrics.rollingPercentile.enabled 执行时是否enable指标的计算和跟踪:

```
            默认true
```

hystrix.command.default.metrics.rollingPercentile.timeInMilliseconds 设置rolling percentile window的时间:

```
           默认60000
```

hystrix.command.default.metrics.rollingPercentile.numBuckets 设置rolling percentile window的numberBuckets:

```
          逻辑同上。默认6
```

hystrix.command.default.metrics.rollingPercentile.bucketSize:

```
            如果bucket size＝100，window＝10s，若这10s里有500次执行，只有最后100次执行会被统计到bucket里去。增加该值会

           增加内存开销以及排序的开销。默认100
```

hystrix.command.default.metrics.healthSnapshot.intervalInMilliseconds 记录health 快照（用来统计成功和错误绿）的间隔:

```
           默认500ms
```

**Request Context 相关参数**

hystrix.command.default.requestCache.enabled 默认true:

```
         需要重载getCacheKey\(\)，返回null时不缓存
```

hystrix.command.default.requestLog.enabled 记录日志到HystrixRequestLog:

```
          默认true
```

**Collapser Properties 相关参数**

hystrix.collapser.default.maxRequestsInBatch 单次批处理的最大请求数:

```
          达到该数量触发批处理，默认Integer.MAX\_VALUE
```

hystrix.collapser.default.timerDelayInMilliseconds 触发批处理的延迟:

```
            也可以为创建批处理的时间＋该值，默认10
```

hystrix.collapser.default.requestCache.enabled :

```
           是否对HystrixCollapser.execute\(\) and HystrixCollapser.queue\(\)的cache，默认true
```

**ThreadPool 相关参数**

```
   线程数默认值10适用于大部分情况（有时可以设置得更小），如果需要设置得更大，那有个基本得公式可以follow：
```

requests per second at peak when healthy × 99th percentile latency in seconds + some breathing room

每秒最大支撑的请求数 **\(99%**平均响应时间 **+** 多余一点缓冲空间**\)**

比如：每秒能处理**1000**个请求，**99%**的请求响应时间是**60ms**，那么公式是：

**1000 \***（**0.060+0.012**）

基本得原则时保持线程池尽可能小，他主要是为了释放压力，防止资源被阻塞。

当一切都是正常的时候，线程池一般仅会有1到2个线程激活来提供服务

hystrix.threadpool.default.coreSize 并发执行的最大线程数:

```
 默认10
```

hystrix.threadpool.default.maxQueueSize BlockingQueue的最大队列数:

```
       当设为－1，会使用SynchronousQueue，值为正时使用LinkedBlcokingQueue。该设置只会在初始化时有效，之后不能修改     

     threadpool的queue size，除非reinitialising thread executor。默认－1。
```

hystrix.threadpool.default.queueSizeRejectionThreshold :

```
         即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝。因为maxQueueSize不能被动态 

          修改，这个参数将允许我们动态设置该值。if maxQueueSize == -1，该字段将不起作用
```

hystrix.threadpool.default.keepAliveTimeMinutes :

```
             如果corePoolSize和maxPoolSize设成一样（默认实现）该设置无效。
```

hystrix.threadpool.default.metrics.rollingStats.timeInMilliseconds 线程池统计指标的时间:

```
            默认10000
```

hystrix.threadpool.default.metrics.rollingStats.numBuckets 将rolling window划分为n个buckets:

```
           默认10
```

除了上述参数之外，来说几个自动生成或者代码决定的几个参数。

HystrixCommandGroupKey:

```
    通过建立一个command的分组，来统一报告，报警，dashboard或者拥有者规范。在页面可以清楚看到。
```

HystrixCommandKey:

```
     表示监控，熔断，统计信息发布，缓存的值表示。
```

HystrixThreadPoolKey:

```
     资源的分配单元。默认就是使用HystrixCommandGroupKey的值。
```

**具体的hystrixCommand相关内容：**

```
     熔断都是通过把执行方法封装成一个hystrixCommand的对象来实现熔断降级等相关的逻辑，同时完成API调用的统计。
```

根据返回值的个数，具体的hystrixCommand有两个子类：单返回值的GenericCommand和多返回值的BatchHystrixCommand。

BatchHystrixCommand是运行HystrixCollapser的执行类，上面也介绍了两种的区别，简单的就是合并小的请求合并成一个请求减少网络开销，通过放回多次的执行结果。HystrixCollapser实用性场景相对较少，下面还是结合GenericCommand的情况来介绍想要的设计。

![](/assets/generalCommand.png)

1）HystrixInvokable： 只是一个声明接口，表示这个是可以invokable的，没有任何方法。

2）HystrixExcutable：是HystrixCommand 和HystrixCollapser的公共接口。可以端可以把command 和 collapse一样对待，放到集合中。

```
方法：public R execute\(\); 同步执行，返回R。
```

@throws HystrixRuntimeException

```
   如果系统出错，没有降级回调，抛出这个异常。

   @throws HystrixBadRequestException
```

这个错误是当请求参数出错的时候，也不代表目标环境出错。

方法： public Future&lt;R&gt; queue\(\);

作为异步执行的方法。

```
   这个方法会把command 本身放线程池的执行队列中，同时放回future对象。


      如果没有配置单独的线程的话，也就是在同一个线程中执行的话，就和同步执行的效果一样。
```

方法： public Observable&lt;R&gt; observe\(\);  
异步执行，返回结果Observable有订阅者有回调的情况。  
饥渴执行模式下，执行queue和

```
      execute是一样的。
```

如果懒模式下，通过toObservale（）得到。

异常多了一个：@throws IllegalStateException

如果执行多次的话，会触发这个。

3）HystrixObservable:

方法:public Observable&lt;R&gt; observe\(\):  
当有订阅回调的时候，异步执行的方法。

```
     方法:public Observable&lt;R&gt; toObservable\(\);
```

如果在懒执行的情况下有订阅者的情况下才执行，如果饥饿加载的情况下可以通过observe（）得到。回调处理的方是，根据线程还是信号量的方式。

4）HystrixInvokableInfo:  
所有那些定义信息的申明    commandKey，groupKey等等。

5）AbstractCommand：  
对属性commandkey等做了实现，同时对调用和生命周期事件都做了简单分装。  
对返回值，缓存，线程和信 号量等的执行了做了初步实现。

6）HystrixCommand  
申明为带有熔断，降级，舱壁等功能的命令，主要是集成AbstractCommand.

7）AbstractHystrixCommand  
 增加了一个commandAction和FallbackAction。

**主线程和执行线程的参数传递**

```
    如果使用信号量策略模式没有这个问题。但是当hystrix中当使用线程池时，就可能需要把主线程的一些参数传递到执行线程中去，比如认证信息，当前用户信息，会话信息等等。
```

在相同线程中，我们往往可以通过ThreadLocal在不同的类中传递参数，让调用变得简单。但是当使用线程模式来实现熔断的时候，会产生新的线程来执行hystrixCommand。

这个时候我们就要使用HystrxiReqestContext来实现不同线程的参数设置。

HystrixRequestContext内部定义了一个静态变量ThreadLocal,每个线程可以获取自己的HystrixRequestContext对象。

一个请求往往由一个tomcat线程处理，所以在该tomcat线程中，HystrixRequestContext对象可以共享。

private static ThreadLocal&lt;HystrixRequestContext&gt; requestVariables = new ThreadLocal&lt;HystrixRequestContext&gt;\(\);

HystrixRequestContext内部是一个ConcurrentHashMap存储请求变量。

ConcurrentHashMap&lt;HystrixRequestVariableDefault&lt;?&gt;, HystrixRequestVariableDefault.LazyInitializer&lt;?&gt;&gt; state = new ConcurrentHashMap&lt;HystrixRequestVariableDefault&lt;?&gt;, HystrixRequestVariableDefault.LazyInitializer&lt;?&gt;&gt;\(\);

HystrixRequestVariableLifecycle-&gt;HystrixRequestVariable-&gt;HystrixRequestVariableDefault-&gt;HystrixLifecycleForwardingRequestVariable

HystrixRequestVariableLifecycle和HystrixRequestVariable定义了一个请求变量，这个请求变量对象的生命周期为在一个请求内。

HystrixRequestVariableDefault为默认实现类。内部他把变量值存储在HystrixRequestContext对象中。key为当前HystrixRequestVariableDefault对象，value为变量真正的值。

public T get\(\) {

```
    if \(HystrixRequestContext.getContextForCurrentThread\(\) == null\) {

        throw new IllegalStateException\(HystrixRequestContext.class.getSimpleName\(\) + ".initializeContext\(\) must be called at the beginning of each request before RequestVariable functionality can be used."\);

    }

    ConcurrentHashMap&lt;HystrixRequestVariableDefault&lt;?&gt;, LazyInitializer&lt;?&gt;&gt; variableMap = HystrixRequestContext.getContextForCurrentThread\(\).state;



    // short-circuit the synchronized path below if we already have the value in the ConcurrentHashMap

    LazyInitializer&lt;?&gt; v = variableMap.get\(this\);

    if \(v != null\) {

        return \(T\) v.get\(\);

    }
```

HystrixLifecycleForwardingRequestVariable只是一个封装对象，内部封装了一个HystrixRequestVariableLifecycle对象。

HystrixRequestVariableHolder定义了一个静态变量，存储所有的HystrixRequestVariable对象，全局共享。

private static ConcurrentHashMap&lt;RVCacheKey, HystrixRequestVariable&lt;?&gt;&gt; requestVariableInstance = new ConcurrentHashMap&lt;RVCacheKey, HystrixRequestVariable&lt;?&gt;&gt;\(\);

RVCacheKey由两部分组成：当前HystrixRequestVariableHolder对象，指定HystrixConcurrencyStrategy对象。

public T get\(HystrixConcurrencyStrategy concurrencyStrategy\) {

RVCacheKey key = new RVCacheKey\(this, concurrencyStrategy\);

如果没有已经存在的HystrixRequestVariable对象，通过HystrixConcurrencyStrategy新建一个。

if \(rvInstance == null\) {

```
        requestVariableInstance.putIfAbsent\(key, concurrencyStrategy.getRequestVariable\(lifeCycleMethods\)\);
```

｝

在执行hystrxiCommand的时候，会从parentContext中进行拷贝。具体实现在com.netflix.hystrix.strategy.concurrency.HystrixContextRunnable

public void run\(\) {

```
    HystrixRequestContext existingState = HystrixRequestContext.getContextForCurrentThread\(\);



    try {

        HystrixRequestContext.setContextOnCurrentThread\(this.parentThreadState\);



        try {

            this.actual.call\(\);

        } catch \(Exception var6\) {

            throw new RuntimeException\(var6\);

        }

    } finally {

        HystrixRequestContext.setContextOnCurrentThread\(existingState\);

    }
```

所以只要在我们的业务线程中，设置相应的变量值默认使用HystrixRequestVariableDefault类型，然后保留这个引用。就可以在子线程中，通过这个引用得到相应的对象。

正如上面介绍的，key就是这个HystrixRequestVariableDefault对象本身，值是你保留设置的对象。

**1）hystrix自带的缓存介绍**

hystix通过调用远端的服务，把结果进行缓存来进一步提升用户体验。但是hytsrxi的缓存是一个相对简单的功能组件，也没有和spring cache整合

而是开发了一套自己的标签和逻辑满足一些简单的缓存需求。

支持三个annotation：

```
 a）CacheKey 

cacheKey的生成逻辑。

在com.netflix.hystrix.contrib.javanica.cache.CacheInvocationContext对应的上下文中调用com.netflix.hystrix.contrib.javanica.cache.HystrixCacheKeyGenerator的

key的生成逻辑。

 b）CacheRemove

   通过com.netflix.hystrix.contrib.javanica.aop.aspectj.HystrixCacheAspect这个切面类实现cache的清理。

 C）CacheResult 

实现缓存结果。
```

具体代码逻辑可以分析com.netflix.hystrix.HystrixRequestCache来看它的具体实现。

public class HystrixRequestCache {

// the String key must be: HystrixRequestCache.prefix + concurrencyStrategy + cacheKey

private final RequestCacheKey rcKey;

private final static ConcurrentHashMap&lt;RequestCacheKey, HystrixRequestCache&gt; caches = new ConcurrentHashMap&lt;RequestCacheKey, HystrixRequestCache&gt;\(\);

private final HystrixConcurrencyStrategy concurrencyStrategy;

private static final HystrixRequestVariableHolder&lt;ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt;&gt; requestVariableForCache = new HystrixRequestVariableHolder&lt;ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt;&gt;\(new HystrixRequestVariableLifecycle&lt;ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt;&gt;\(\) {

@Override  
        public ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt; initialValue\(\) {

```
                                     return new ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt;\(\);
    }
```

@Override  
        public void shutdown\(ConcurrentHashMap&lt;ValueCacheKey, HystrixCachedObservable&lt;?&gt;&gt; value\) {

```
                      // nothing to shutdown
    }
}\);



       // ... 省略其他属性与方法
```

}

rcKey 属性：

```
 请求缓存 KEY由三部分组成 ：type ：类型，分成 Collapser 、Command 两种。
                       key ：键。当类型为 Command 时，使用 com.netflix.hystrix.HystrixCommandKey 。当类型为 Collapser 时，使用 com.netflix.hystrix.HystrixCollapserKey 。
                       concurrencyStrategy：并发策略。
```

caches 静态属性：HystrixRequestCache 同时也是 HystrixRequestCache 的管理器。缓存的存放地方。

获得 HystrixRequestCache 。若不存在，根据 RequestCacheKey 和 HystrixConcurrencyStrategy 创建 HystrixRequestCache ，考虑并发的情况下，添加到 caches。

HystrixRequestVariableHolder 可以理解成一个 Key 为 HystrixConcurrencyStrategy ，Value 为 ConcurrentHashMap 的一个 Map 。所以，requestVariableForCache 可以看成一个两层的 Map。为什么可以这么理解？HystrixRequestVariableHolder 提供 \#get\(HystrixConcurrencyStrategy\) 的方法。

HystrixRequestVariableHolder\(HystrixRequestVariableLifecycle\) 构造方法，传递参数 HystrixRequestVariableLifecycle 。  
HystrixRequestVariableLifecycle ：定义 HystrixRequestVariable 的创建\( \#initialValue\(\) \)和关闭\(\#shutdown\(...\)\)的接口方法。

当我们现在了自己的cache体统的时候，或者说已经和spring cache整合好了ehCache或者redis等等的时候，更人建议就不要用这个cache。理由是它比较简单，而且如果多套cache内容并且也存在同步的问题和浪费存储空间。

如果真要用的话， 就要实现想要的时间来监听cache事件对两种进行同步。

**2）与springCache的整合**

```
  和spring cache的整合很简单，只要使用spring cache那一条annotation就可以。具体的实现在缓存章节有具体的介绍。
```

这里主要是要处理一种情况，当远端报错也就是5XX错误。如果该远程调用方法设置了fallback方法，那么还是会有一个正常的

返回值。这种情况下，spring cache的默认逻辑会把结果缓存起来。很明显这种情况下，我们一般不希望缓存这个结果。这样我们

需要改变spring cache的默认行为。

spring cache是通过org.springframework.cache.interceptor.CacheInterceptor这个拦截器实现的。

在org.springframework.cache.annotation.ProxyCachingConfiguration来装配的。代码如下：

```
@Bean

@Role\(2\)

public CacheInterceptor cacheInterceptor\(\) {

    CacheInterceptor interceptor = new CacheInterceptor\(\);

    interceptor.setCacheOperationSources\(new CacheOperationSource\[\]{this.cacheOperationSource\(\)}\);

    if\(this.cacheResolver != null\) {

        interceptor.setCacheResolver\(this.cacheResolver\);

    } else if\(this.cacheManager != null\) {

        interceptor.setCacheManager\(this.cacheManager\);

    }



    if\(this.keyGenerator != null\) {

        interceptor.setKeyGenerator\(this.keyGenerator\);

    }



    if\(this.errorHandler != null\) {

        interceptor.setErrorHandler\(this.errorHandler\);

    }



    return interceptor;

}
```

所以我们可以通过重写cacheInterceptor 中的doPut方法即可。

public void doPut\(Cache cache, Object key, Object result\)

逻辑就很清晰了，当服务段出错的时候通过统一的错误处理逻辑在HystrixRequestVariable中增加一个status放入到HystrixRequestContext中。

示例代码如下：

@Override

```
public void doPut\(Cache cache, Object key, Object result\) {

    LOGGER.debug\("before doPut...."\);

    for \(HystrixRequestVariableDefault&lt;EbaoFeignContext&gt; currentHystrixContextVariable : EbaoFeignContext.getHystrixRequestVariableList\(\)\) {

        if \(currentHystrixContextVariable.get\(\) != null\) {

            if \(currentHystrixContextVariable.get\(\).isExecutionSuccessFlg\(\)\) {

                LOGGER.debug\("The rest calling is successful. Putting the cache."\);

                LOGGER.debug\("key:{}", key\);

                LOGGER.debug\("result:{}", result\);

                super.doPut\(cache, key, result\);

            }

        }

    }

}
```

这样我们可以使用spring cache的功能了。

3）统一的远端调用异常处理

```
这里为什么说远端，因为http协议我们知道常见的两大类错粗一个是4XX是指调用端的错误，BadRequest等等
```

还有一类错误是5XX的错误，是指被调用的服务出现了异常。这里的统一错误就是指的这一类。

比如在这个时候，我们要写一些特殊的log包括即是有fallback方法等也通知不能缓存数据等都可以在这个代码中实现。

HystrixRequest上下文中标记一下这个请求失败了，让客户端来消费这个结果。

public class HystrixRemoteErrorDecoder extends ErrorDecoder.Default {

```
private static final Logger LOGGER = LoggerFactory.getLogger\(\);



@Override

public Exception decode\(String methodKey, Response response\) {

LOGGER.error\(exception.getMessage\(\), exception\);

    // mark the call as fail.

     for \(HystrixRequestVariableDefault&lt;FeignContext&gt; currentHystrixContextVariable :         FeignContext.getHystrixRequestVariableList\(\)\) {

        if \(currentHystrixContextVariable.get\(\) != null\) {

            currentHystrixContextVariable.get\(\).setExecutionSuccessFlg\(false\);

        }

    }

    return exception;



}
```

}

4）熔断监控

1）SSE

熔断的官方现实是使用SSE实现的实时刷新。

SSE （ Server-sent Events ）是 WebSocket 的一种轻量代替方案，使用 HTTP 协议。

严格地说，HTTP 协议是没有办法做服务器推送的，但是当服务器向客户端声明接下来要发送流信息时，客户端就会保持连接打开，SSE 使用的就是这种原理。

SSE 和 WebSocket 做的是同一件事情。当你需要用新数据局部更新网络应用时，SSE 可以做到不需要用户执行任何操作，便可以完成。

举例我们要做一个统计系统的管理后台，我们想知道统计数据的实时情况。类似这种更新频繁、 低延迟的场景，SSE 可以完全满足。

SSE 是单向通道，只能服务器向客户端发送消息，如果客户端需要向服务器发送消息，则需要一个新的 HTTP 请求。

这对比 WebSocket 的双工通道来说，会有更大的开销。这么一来的话就会存在一个「什么时候才需要关心这个差异？」的问题，

如果平均每秒会向服务器发送一次消息的话，那应该选择 WebSocket。如果一分钟仅 5 - 6 次的话，其实这个差异并不大。

SSE技术的优势：

实现一个完整的服务仅需要少量的代码和标准的restful基本一致；

是协议层的，可以用任何一种服务端语言中使用；

基于 HTTP ／ HTTPS 协议，可以直接运行于现有的代理服务器和认证技术。

服务器首先向客户端声明接下来发送的是事件流（ text/event-stream ）类型的数据，然后就可以向客户端多次发送消息。

事件流是一个简单的文本流，仅支持 UTF-8 格式的编码。每条消息以一个空行作为分隔符。

在规范中为消息定义了 4 个字段：

event 消息的事件类型。客户端收到消息时，会在当前的 EventSource 对象上触发一个事件，这个事件的名称就是这个字段的值，如果消息没有这个字段，客户端的 EventSource 对象就会触发默认的 message 事件。

id 这条消息的 ID。客户端接收到消息后，会把这个 ID 作为内部属性 Last-Event-ID，在断开重连 成功后，会把 Last-Event-ID 发送给服务器。

data 消息的数据字段。 客户端会把这个字段解析为字符串，如果一条消息有多个 data 字段，客户端会自动用换行符 连接成一个字符串。

retry 指定客户端重连的时间。只接受整数，单位是毫秒。如果这个值不是整数则会被自动忽略。

SSE 如何保证数据完整性

客户端在每次接收到消息时，会把消息的 id 字段作为内部属性 Last-Event-ID 储存起来。

SSE 默认支持断线重连机制，在连接断开时会 触发 EventSource 的 error 事件，同时自动重连。再次连接成功时 EventSource 会把 Last-Event-ID 属性作为请求头发送给服务器，这样服务器就可以根据这个 Last-Event-ID 作出相应的处理。

这里需要注意的是，id 字段不是必须的，服务器有可能不会在消息中带上 id 字段，这样子客户端就不会存在 Last-Event-ID 这个属性。所以为了保证数据可靠，我们需要在每条消息上带上 id 字段。

向下兼容

早些时候，为了实现数据实时更新最常见的方法就是轮询。

轮询是以一个固定频率向服务器发送请求，服务器在有 数据更新时 返回新的数据，以此来管理数据的更新。这种轮询的方式不但开销大，而且更新的效率和频率有关，也不能达到及时更新的目的。

接着便出现了长轮询的方式：客户端向服务器发送请求之后，服务器会暂时把请求挂起，等到有数据更新时再返回最新的数据给客户端，客户端在接收到新的消息后再向服务器发送请求。与常规轮询的不同之处是：数据可以做到实时更新，可以减少不必要的开销。

这里有一个「选择长轮询还是常规轮询？」的命题，长轮询是不是总比常规轮询占有优势？我们可以从带宽占用的角度分析，如果一个程序数据更新太过频繁，假设每秒 2 次更新，如果使用长轮询的话每分钟要发送 120 次 HTTP 请求。如果使用常规轮询，每 5 秒发送一次请求的话， 一分钟才 20 次，从这里看，常规轮询更占有优势。

长轮询和 SSE 最关键的区别在于，每一次数据更新都需要一次 HTTP 请求。和 WebSocket 还有 SSE 一样，长轮询也会 占用一个 socket。在数据更新效率上和 SSE 差不多，一有数据更新就能检测到。加上所有浏览器都支持，是一个不错的 SSE 替代方案。

javascript 创建SSE连接代码式样：

'use strict';

function connectSSE\(\) {

if \(window.EventSource\) {

```
const source = new EventSource\('http://localhost:2000'\);

let reconnectTimeout;



source.addEventListener\('open', \(\) =&gt; {

  console.log\('Connected'\);

  clearTimeout\(reconnectTimeout\);

}, false\);



source.addEventListener\('pause', e =&gt; {

  source.close\(\);

  const reconnectTime = +e.data;

  const currentTime = +new Date\(\);

  reconnectTimeout = setTimeout\(\(\) =&gt; {

    connectSSE\(\);

  }, reconnectTime - currentTime\);

}, false\);
```

} else {

```
console.error\('Your browser doesn\'t support SSE'\);
```

}

}

connectSSE\(\);

javascript创建SSE相应代码式样：

'use strict';

if \(window.EventSource\) {

// 创建 EventSource 对象连接服务器

const source = new EventSource\('[http://localhost:2000'\](http://localhost:2000'\)\);

// 连接成功后会触发 open 事件

source.addEventListener\('open', \(\) =&gt; {

```
console.log\('Connected'\);
```

}, false\);

// 服务器发送信息到客户端时，如果没有 event 字段，默认会触发 message 事件

source.addEventListener\('message', e =&gt; {

    console.log\(\`data: ${e.data}\`\);

}, false\);

// 自定义 EventHandler，在收到 event 字段为 slide 的消息时触发

source.addEventListener\('slide', e =&gt; {

    console.log\(\`data: ${e.data}\`\); // =&gt; data: 7

}, false\);

// 连接异常时会触发 error 事件并自动重连

source.addEventListener\('error', e =&gt; {

```
if \(e.target.readyState === EventSource.CLOSED\) {

  console.log\('Disconnected'\);

} else if \(e.target.readyState === EventSource.CONNECTING\) {

  console.log\('Connecting...'\);

}
```

}, false\);

} else {

console.error\('Your browser doesn\'t support SSE'\);

}

用javascript实现SSE服务端代码的式样：

'use strict';

const http = require\('http'\);

http.createServer\(\(req, res\) =&gt; {

// 服务器声明接下来发送的是事件流

res.writeHead\(200, {

```
'Content-Type': 'text/event-stream',

'Cache-Control': 'no-cache',

'Connection': 'keep-alive',

'Access-Control-Allow-Origin': '\*',
```

}\);

// 发送消息

setInterval\(\(\) =&gt; {

    res.write\('event: slide\n'\); // 事件类型

    res.write\(\`id: ${+new Date\(\)}\n\`\); // 消息 ID

    res.write\('data: 7\n'\); // 消息数据

    res.write\('retry: 10000\n'\); // 重连时间

    res.write\('\n\n'\); // 消息结束

}, 3000\);

// 发送注释保持长连接

setInterval\(\(\) =&gt; {

```
res.write\(': \n\n'\);
```

}, 12000\);

}\).listen\(2000\);

一个很有意思的地方是，规范中规定以冒号开头的消息都会被当作注释，一条普通的注释（:\n\n）对于服务器来说只占 5 个字符，但是发送到客户端上的时候不会触发任何事件，这对客户端来说是非常友好的。所以注释一般被用于维持服务器和客户端的长连接。

在hystrix的springMVC版本的代码实现

com.netflix.hystrix.contrib.sample.stream.HystrixMetricsStreamServlet

![](/assets/屏幕快照 2018-03-31 下午8.12.30.png)

private void handleRequest\(HttpServletRequest request, HttpServletResponse response\) throws ServletException, IOException {

```
    final AtomicBoolean moreDataWillBeSent = new AtomicBoolean\(true\);

    Subscription sampleSubscription = null;

    int numberConnections = this.incrementAndGetCurrentConcurrentConnections\(\);



    try {

        int maxNumberConnectionsAllowed = this.getMaxNumberConcurrentConnectionsAllowed\(\);

        if \(numberConnections &gt; maxNumberConnectionsAllowed\) {

            response.sendError\(503, "MaxConcurrentConnections reached: " + maxNumberConnectionsAllowed\);

        } else {

            response.setHeader\("Content-Type", "text/event-stream;charset=UTF-8"\);

            response.setHeader\("Cache-Control", "no-cache, no-store, max-age=0, must-revalidate"\);

            response.setHeader\("Pragma", "no-cache"\);
```

**annotation和FeignClient Bean熔断设置：**

在微服务内部调用主要就是通过RestTemplae和FeignClient封装的restful Bean来实现远程调用。

熔断也主要是针对远端请求的。当然其他的资源也可以，只要是不稳定的或可能成为系统的不稳定因素都可以

通过这种方式来进行隔离，让大部分的服务总是满足预期。

在系统中的不稳定资源有很多，根据不稳定程度划分等级。可以现在最不稳定和影响最深的点入手。

常常不稳定的有DB连接，中间件的连接包括消息中间件，搜索引擎，工作流等等。

但是往往最坏的影响还是系统雪崩，当一个系统不可用的时候，其他依赖的服务纷纷不可用造成web容器的假死等。

所以最初使用熔断的还是微服务之间的远程调用，因为它们是系统的血脉布满整个系统是可用性的重中之重。

为了统一两种使用方式，那么groupKey，commandKey，ThreadPoolKey要保持一致。

最方式的方法就是让GroupKey和ThreadPoolKey要调用的微服务名字保持一致，CommandKey和url保持一致。

这样无论是用什么方法，资源分配和统计都是一个合理真实的结果，而不是使用系统默认用className和method。

在微服务的远程调用中，restTemplate和FeignClien两种。下面一一说明使用的区别。

1）Annotation的熔断设置

参数配置

可以通过查看源码的方式，看到具体有哪些配置，主要有：GroupKey，CommandKey，fallback，TheadPoolKey等等。

fallback：必须当前类中的方法的方法名称，降级调用的方法

GroupKey：统计时候用的组名

CommandKey：熔断单位名名称

ThreadPoolKey：线程组名称，默认使用GroupKey

代码具体实现

通过在方法上加上Annotation标签：HystrixCommand或者HystrixCollapser，通过拦截器实现对方法调用的封装

com.netflix.hystrix.contrib.javanica.aop.aspectj.HystrixCommandAspect。如果这个拦截器里面要求

不满足我们的需求，可以实现自己来实现。对应的配置文件如下：

org.springframework.cloud.netflix.hystrix.HystrixCircuitBreakerConfiguration

@Bean

```
public HystrixCommandAspect hystrixCommandAspect\(\) {

    return new HystrixCommandAspect\(\);
```

}

等我们自己实现了拦截器之后，然后重新在spring.factories文件中

org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker=

com.\*\*\*.customization.HystrixCircuitBreakerConfiguration

甚至可以自己来实现具体的HystrixCommand替换系统的genericCommand

_**Annotation实现客制化**_

正如上面说道有时候需要自己根据规则来定义GroupKey，CommandKey等，甚至还需要自定义Command。

RestTemplate使用比较灵活，很多时候URL部分都是根据参数拼接出来的，所以这些属性需要支持表示的形式。

就使用这个例子来客制化自己的SetterFactory。

创建自己的HystrixCommand集成自@HystrixCommand

public class ConcreteHystrixCommand implements HystrixCommand {

```
private String groupKey;

private String commandKey;

private String threadPoolKey;

private String fallbackMethod;

private HystrixProperty\[\] commandProperties;

private HystrixProperty\[\] threadPoolProperties;

private Class&lt;? extends Throwable&gt;\[\] ignoreExceptions;

private ObservableExecutionMode observableExecutionMode;

private HystrixException\[\] raiseHystrixExceptions;

private String defaultFallback;



public void setGroupkey\(String groupkey\) {

    this.groupKey = groupkey;

}



public void setCommandKey\(String commandKey\) {

    this.commandKey = commandKey;

}



public void setThreadPoolKey\(String threadPoolKey\) {

    this.threadPoolKey = threadPoolKey;

}



public void setFallbackMethod\(String fallbackMethod\) {

    this.fallbackMethod = fallbackMethod;

}



public void setCommandProperties\(HystrixProperty\[\] commandProperties\) {

    this.commandProperties = commandProperties;

}



public void setThreadPoolProperties\(HystrixProperty\[\] threadPoolProperties\) {

    this.threadPoolProperties = threadPoolProperties;

}



public void setIgnoreExceptions\(Class&lt;? extends Throwable&gt;\[\] ignoreExceptions\) {

    this.ignoreExceptions = ignoreExceptions;

}



public void setObservableExecutionMode\(ObservableExecutionMode observableExecutionMode\) {

    this.observableExecutionMode = observableExecutionMode;

}



public void setRaiseHystrixExceptions\(HystrixException\[\] raiseHystrixExceptions\) {

    this.raiseHystrixExceptions = raiseHystrixExceptions;

}



public void setDefaultFallback\(String defaultFallback\) {

    this.defaultFallback = defaultFallback;

}



public void setAnnotationType\(Class&lt;? extends Annotation&gt; annotationType\) {

    this.annotationType = annotationType;

}



private Class&lt;? extends Annotation&gt; annotationType;



@Override

public String groupKey\(\) {

    return this.groupKey;

}



@Override

public String commandKey\(\) {

    return this.commandKey;

}



@Override

public String threadPoolKey\(\) {

    return this.threadPoolKey;

}



@Override

public String fallbackMethod\(\) {

    return this.fallbackMethod;

}



@Override

public HystrixProperty\[\] commandProperties\(\) {

    return this.commandProperties;

}



@Override

public HystrixProperty\[\] threadPoolProperties\(\) {

    return this.threadPoolProperties;

}



@Override

public Class&lt;? extends Throwable&gt;\[\] ignoreExceptions\(\) {

    return this.ignoreExceptions;

}



@Override

public ObservableExecutionMode observableExecutionMode\(\) {

    return this.observableExecutionMode;

}



@Override

public HystrixException\[\] raiseHystrixExceptions\(\) {

    return this.raiseHystrixExceptions;

}



@Override

public String defaultFallback\(\) {

    return this.defaultFallback;

}



@Override

public Class&lt;? extends Annotation&gt; annotationType\(\) {

    return this.annotationType;

}





public void copyPropertiesFromAnnoation\(HystrixCommand hystrixCommand\){

    this.groupKey = hystrixCommand.groupKey\(\);

    this.commandKey = hystrixCommand.commandKey\(\);

    this.threadPoolKey = hystrixCommand.threadPoolKey\(\);

    this.fallbackMethod = hystrixCommand.fallbackMethod\(\);

    this.commandProperties = hystrixCommand.commandProperties\(\);

    this.threadPoolProperties = hystrixCommand.threadPoolProperties\(\);

    this.ignoreExceptions = hystrixCommand.ignoreExceptions\(\);

    this.observableExecutionMode = hystrixCommand.observableExecutionMode\(\);

    this.raiseHystrixExceptions = hystrixCommand.raiseHystrixExceptions\(\);

    this.defaultFallback = hystrixCommand.defaultFallback\(\);

}
```

在拦截器中去重写CommandMetaHolderFactory中的create方法

private static class CommandMetaHolderFactory extends MetaHolderFactory {

```
    @Override

    public MetaHolder create\(Object proxy, Method method, Object obj, Object\[\] args, final ProceedingJoinPoint joinPoint\) {

        HystrixCommand hystrixCommand = method.getAnnotation\(HystrixCommand.class\);

        ConcreteHystrixCommand concreteHystrixCommand = new ConcreteHystrixCommand\(\);

        concreteHystrixCommand.copyPropertiesFromAnnoation\(hystrixCommand\);

        concreteHystrixCommand.setGroupkey\(FeignHystrixSetterFactory.getStrValueOfExpression\(obj, method, args, concreteHystrixCommand.groupKey\(\)\)\);

        Assert.isNotEmpty\(concreteHystrixCommand.groupKey\(\), "The group key is null."\);

        String commandKey = FeignHystrixSetterFactory.getStrValueOfExpression\(obj, method, args, concreteHystrixCommand.commandKey\(\)\);

        Assert.isNotEmpty\(commandKey, "The command key is null."\);

        String absCommandKey = commandKey.startsWith\("/"\) ? \(concreteHystrixCommand.groupKey\(\) + commandKey\) : \(concreteHystrixCommand.groupKey\(\) + "/" + commandKey\);

        concreteHystrixCommand.setCommandKey\(absCommandKey\);

        ExecutionType executionType = ExecutionType.getExecutionType\(method.getReturnType\(\)\);

        MetaHolder.Builder builder = metaHolderBuilder\(proxy, method, obj, args, joinPoint\);

        if \(isCompileWeaving\(\)\) {

            builder.ajcMethod\(getAjcMethodFromTarget\(joinPoint\)\);

        }

        return builder.defaultCommandKey\(method.getName\(\)\)

                .hystrixCommand\(concreteHystrixCommand\)

                .observableExecutionMode\(hystrixCommand.observableExecutionMode\(\)\)

                .executionType\(executionType\)

                .observable\(ExecutionType.OBSERVABLE == executionType\)

                .build\(\);

    }

}
```

其中表示式的支持就通过使用SPEL做一个简单的分装就可以实现，这里就不一一细说了。

其他需要客制化的地方，可以按照这个来处理。

2）FeignClient Bean的hystrix设置

相对来说配置要简单一些，而且做了更多的默认配置。

GroupKey：默认使用当前类名

CommandKey：默认使用当前方法名称

还有fallback 和fallbackFactory两个属性。

fallback与Annotation的要求不一样，它要制定一个class而且根据是spring的一个bean。

当设置了fallback的时候，设置fallbackFactory是无效的。

FallbackFactory当，实现它的create方法对错误统一处理。

更多的配置FeigCLient可以指定自己的Configuration文件进行配置参考

org.springframework.cloud.netflix.feign.FeignClientsConfiguration.

@Bean

```
    @Scope\("prototype"\)

    @ConditionalOnMissingBean

    @ConditionalOnProperty\(

        name = {"feign.hystrix.enabled"},

        matchIfMissing = false

    \)

    public Builder feignHystrixBuilder\(\) {

        return HystrixFeign.builder\(\);

    }
```

具体可以配置的项目在HystrixFeign里面有详细的定义。

_**FeignClient客制化HystrixCommad**_

在FeignClient里面要实现客制化，也通过上面那个GroupKey和CommandKey的例子。

FeignClient是通过SettorFactory来创建HystrixCommand的，所以自然要通过

重新定义SetterFactory，通过替换到原来的default实现才能达到目的。

代码式例如下：

public class FeignHystrixSetterFactory implements SetterFactory {

```
private static final Logger LOGGER = LoggerFactory.getLogger\(\);



@Override

public HystrixCommand.Setter create\(Target&lt;?&gt; target, Method method\) {

    String serverName = target.type\(\).getAnnotation\(FeignClient.class\).name\(\);

    String path = target.type\(\).getAnnotation\(FeignClient.class\).path\(\);

    String url = getUriByMethod\(method\);

    LOGGER.debug\("class :{}", target.getClass\(\)\);

    LOGGER.debug\("method :{}", method.getName\(\)\);

    LOGGER.debug\("serverName :{}", serverName\);

    Assert.isNotEmpty\(path + url, "The url is null."\);

    String absUrl = \(path + url\).startsWith\("/"\) ? \(serverName + path + url\) : \(serverName + "/" + path + url\);

    LOGGER.debug\("url :{}", absUrl\);

    return HystrixCommand.Setter

            .withGroupKey\(HystrixCommandGroupKey.Factory.asKey\(serverName\)\)

            .andCommandKey\(HystrixCommandKey.Factory.asKey\(absUrl\)\)

            .andThreadPoolKey\(HystrixThreadPoolKey.Factory.asKey\(serverName\)\);



}
```

//TODO _**参数热替换**_



