### JDK中的线程池支持
```Executor``` 框架
```Executors``` 工具类默认提供各种线程池的构建
- 重点：java8开始提供了一些新的线程池类型,用于并行处理,待学习

```ThreadPoolExecutor``` 是官方提供的线程池的本质
```
new ThreaPoolExecutor(
int corePoolSize, //指定线程池中的线程数量
int maximumPoolSize, //指定线程的最大数量
long keepAliveTime, // 指定大于corePoolSize的多余线程的存活时间
TimeUnit unit, //时间单位
BlockingQueue<Runnable> workQueue, // 任务队列,已提交未执行的任务
ThreadFactory threadFactory, //线程工厂类,一般用默认
RejectedExecutionHandler handler //拒绝策略,任务过多时的拒绝处理
);
```
- 其中workQueue分为以下几类
```
zhilySynchronousQueue //直接提交队列,没有容量,任务不会真实保存,每个任务的提交需要一个任务的完成,即总是将新任务提交给线程执行,没有空闲的就创建新线程,如果达到最大值就执行拒绝策略;

ArrayBlockingQueue //有界任务队列,有容量且必须提供该参数来构造,当前线程数小于corePoolSize时,优先创建新线程;达到corePoolSize后,则优先存储任务,直到装满队列,再根据maximumPoolSize创建新线程,如果线程数大于最大线程数,则执行拒绝策略。有界任务队列试图维持线程池中的线程数在corePoolSize的数量。

LinkedBlockingQueue //无界任务队列,没有容量上限,当前线程数小于corePoolSize时,优先创建新线程;达到corePoolSize后不再增加,此时如果持续有新任务加入而没有空闲线程，则存储任务,直至资源耗尽。

PriorityBlockingQueue //优先任务队列,通过优先级控制任务的执行先后顺序,特殊的无界队列。
```

```newFixedThreadPool和newSingleThreaPool``` 使用```LinkedBlockingQueue```,他们的核心线程数和最大线程数相同,因为是固定大小的线程池;

```newCachedThreadPool``` 则使用```SynchronousQueue```,它的核心线程数是0,最大线程数是Integer的最大值,线程存活时间是60秒。

### ThreadPoolExecutor的执行逻辑
- 简单图解
```
任务提交->小于coresize->分配线程执行
        |
        ->大于coresize->提交等待队列->成功->等待执行
                                    |
                                    ->失败->提交线程池->达到max线程提交失败->拒绝执行 
                                                     |
                                                     ->未达到最大线程数提交成功->分配线程执行
                                    
```

- 拒绝策略默认提供了4种
```
AbortPolicy //直接异常,阻止系统正常工作

CallerRunsPolicy //只要没有关闭线程池,就在调用者所在线程执行任务,不会真正丢弃任务,但是会影响任务提交线程的性能

DiscardOledestPolicy //丢弃一个最老的任务(即将被执行),再尝试提交新任务

DiscardPolicy //丢弃无法处理的任务,允许任务丢失时的最佳选择(暂时)

以上4种策略均实现了RejectExecutionHandler接口,如果不满足需求,则可以自定义拒绝策略,实现该接口的rejectExecution(Runnable r,ThreadPoolExecutor executor)方法;
其中r为请求执行的任务,executor为当前的线程池
```

### ThreadPoolExecutor的扩展性
- 继承并覆盖以下三个方法来扩展
```
beforeExecute();
afterExecute();
terminated();
```
- 还可以覆盖execute方法,封装提交的Runnable任务,来扩展更多的功能,譬如增加堆栈信息的追踪等。
- 合理设置线程池线程数量,估计的公式:Nthreads = Ncpu*Ucpu*(1+W/C)
```
Nthreads = Ncpu*Ucpu*(1+W/C);
Ncpu = cpu数量
Ucpu = cpu使用率
W/C = 等待时间和计算时间的比率

--by 《Java Concurrency in Practice》
```

### ThreaLocal 保存只有当前线程能访问的资源
- 但是,如果每个线程分配到的都是同一个对象,依然会有线程安全。所以,ThreaLocal适合可以为每一个线程分配独立副本资源的场景下使用(譬如同一个实例内的成员属性需要线程隔离的情况——自己推测的,未证实!)

- 其本质是每个线程实例内部持有一个ThreadLocal.ThreadLocalMap(这里有两种一个是threadLocals,一个是inheritableThreadLocals,后者可以获得继承效果获取父类的数据)的Map对象,以ThreadLocal实例引用作为key,实例的值作为value!

### 这里之前理解有误！
- ThreadLocal中的ThreadLocalMap是一个类似于WeakHashMap的存在,内部的ThreadLoacal引用(即key)都是弱引用,由于弱引用的机制,当外部ThreadLoacal引用(强引用)回收时,内部的key就变成null了,所以这些线程持有的对象就都被回收了。
- 使用ThreadLocal之后,如果还有线程池的话,处理不当可能会有内存泄漏的情况,因为对象都是保存在线程中的,线程不退出,对象不会被回收。