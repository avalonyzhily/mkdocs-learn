### ScheduledExecutorService:
1. 用作定时任务,相比Timer的单线程执行任务更好;
2. java并发包下的类;

### 重入锁和synchronized关键字:

现在性能上差不多,但是重入锁更灵活;

基础用法
```
ReentrantLock lock = new ReentrantLock();
lock.lock();
......
lock.unlock();
```
优先响应中断的锁
```
lock.lockInterruption()
```
带等待时限
```
if(lock.tryLock(5,TimeUnit.SECONDS))//若tryLock()不带时限,则不等待
......
if(lock.isHeldByCurrentThread())
lock.unlock();
```

synchronized关键字的锁是非公平锁,但是重入锁允许设置公平锁
```new ReentrantLock(boolean fair)```
但是公平所的性能上有额外开销,一般没有必要使用。

重入锁里的 wait和notify——```await 和 signal```,```Condition condition=lock.newCondition()```
ArrayBlockQueue中有应用(java内部代码的应用很多)

### Semaphore 信号量
信号量允许多个线程同时访问
```
Semaphore sema = new Semaphore(int i[,boolean fair]);//i是信号量准入数,即线程数,还可以设置是否为公平锁
sema.acquire(); //获取许可
sema.tryAcquire([long time,TimeUnit tu]);同tryLock
sema.release(); //释放许可
```


### ReentrantReadWriteLock 读写锁
读写分离锁,让读操作并行,写操作阻塞串行
```
ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
lock.readLock(); //读锁互不排斥,读写锁互相排斥
lock.writeLock; //写锁互相排斥
```

### CountDownLatch 倒计时器
```
CountDownLatch edl = new CountDownLatch(int i);
edl.countDown();//计数操作,完成一项就减一
edl.await();//等待计数结束,检查所有线程任务是否执行完毕
```

### CyclicBarrier 循环栅栏
可复用倒计时器,对比倒计时器,不需要显式的CountDown()
```
CyclicBarrier eb = new CyclicBarrier(int i,Runnable action);//i是参与线程的总数,action是完成一轮计数之后,触发执行的方法
eb.await();//等待计数结束,检查所有线程任务是否执行完毕,可以重复调用,进入下一次计数
BrokenBarrierException;表示当前的循环栅栏已经破坏,无法完成继续等待
```

### LockSupport 线程阻塞工具类
不需要获取锁(内部采用类似信号量的机制),不会抛出中断异常(但是可以获取中断标识),不会出现unpark在park之前,导致死锁的情况
```
LockSupport.park();//阻塞
LockSupport.unpark();//解除阻塞
```

### JDK中的线程池支持
```Executor``` 框架
```Executors``` 工具类默认提供各种线程池的构建
# 重点：java8开始提供了一些新的线程池类型,用于并行处理,待学习

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
其中workQueue分为以下几类
```
SynchronousQueue //直接提交队列,没有容量,任务不会真实保存,每个任务的提交需要一个任务的完成,即总是将新任务提交给线程执行,没有空闲的就创建新线程,如果达到最大值就执行拒绝策略;

ArrayBlockingQueue //有界任务队列,有容量且必须提供该参数来构造,当前线程数小于corePoolSize时,优先创建新线程;达到corePoolSize后,则优先存储任务,直到装满队列,再根据maximumPoolSize创建新线程,如果线程数大于最大线程数,则执行拒绝策略。有界任务队列试图维持线程池中的线程数在corePoolSize的数量。

LinkedBlockingQueue //无界任务队列,没有容量上限,当前线程数小于corePoolSize时,优先创建新线程;达到corePoolSize后不再增加,此时如果持续有新任务加入而没有空闲线程，则存储任务,直至资源耗尽。

PriorityBlockingQueue //优先任务队列,通过优先级控制任务的执行先后顺序,特殊的无界队列。
```

```newFixedThreadPool和newSingleThreaPool``` 使用```LinkedBlockingQueue```,他们的核心线程数和最大线程数相同,因为是固定大小的线程池;

```newCachedThreadPool``` 则使用```SynchronousQueue```,它的核心线程数是0,最大线程数是Integer的最大值,线程存活时间是60秒。

### ThreadPoolExecutor的执行逻辑
```
任务提交->小于coresize->分配线程执行
        |
        ->大于coresize->提交等待队列->成功->等待执行
                                    |
                                    ->失败->提交线程池->达到max线程提交失败->拒绝执行 
                                                     |
                                                     ->未达到最大线程数提交成功->分配线程执行
                                    
```

拒绝策略默认提供了4种
```
AbortPolicy //直接异常,阻止系统正常工作

CallerRunsPolicy //只要没有关闭线程池,就在调用者所在线程执行任务,不会真正丢弃任务,但是会影响任务提交线程的性能

DiscardOledestPolicy //丢弃一个最老的任务(即将被执行),再尝试提交新任务

DiscardPolicy //丢弃无法处理的任务,允许任务丢失时的最佳选择(暂时)

以上4种策略均实现了RejectExecutionHandler接口,如果不满足需求,则可以自定义拒绝策略,实现该接口的rejectExecution(Runnable r,ThreadPoolExecutor executor)方法;
其中r为请求执行的任务,executor为当前的线程池
```

### ThreadPoolExecutor的扩展性
继承并覆盖以下三个方法来扩展
```
beforeExecute();
afterExecute();
terminated();
```
还可以覆盖execute方法,封装提交的Runnable任务,来扩展更多的功能,譬如增加堆栈信息的追踪等。


合理设置线程池线程数量,估计的公式:Nthreads = Ncpu*Ucpu*(1+W/C)
```
Nthreads = Ncpu*Ucpu*(1+W/C);
Ncpu = cpu数量
Ucpu = cpu使用率
W/C = 等待时间和计算时间的比率

--by 《Java Concurrency in Practice》
```

### 锁优化的几个维度

时间——减少所持有的时间
空间——缩小锁控制的粒度,可以分割数据结构(譬如ConcurrentHashMap分段锁),也可以分割系统功能(譬如读写分离锁);

锁分离——LinkedBlockingQueue的实现,使用Condition+ReentrantLock,在队列的首位分别使用分离的takeLock和putLock;这里与ArrayBlockingQueue的实现中只有一把锁不一样。

锁粗化——这点与缩小锁控制粒度有点冲突,但并不违背,主要场景是连续对同一锁的请求和释放操作是可以合并的。

### ConcurrentHashMap 
为了减小锁的粒度,```ConcurrentHashMap``` 默认分成16个段,通过hashcode来确认插入键值对所在段位,然后对段位加锁进行操作,因此当需要获得全局锁时(访问全局信息,譬如size()方法)会增加开销,性能会低于同步的HashMap。


### JVM的锁优化 都是为了避免线程被挂起

锁偏向——对于同一线程多次请求锁,启用锁偏向模式,则在二次请求锁时无需同步操作;但锁竞争激烈时,极少有同一线程二次请求的情况,锁偏向模式失效,不如不启用。JVM参数：-XX:+UseBiasedLocking=true/false

轻量级锁——偏向锁失败情况下,不会立即挂起线程,通过对象头部指针指向持有锁线程堆栈内部来判断加锁是否成功,成功则进入临界区;失败则证明锁已经被别的线程持有,当前线程的锁请求变为重量级锁(这里理解不是很清晰,轻量级锁和重量级锁差别有多少,锁请求的完整过程是怎样的?)

自旋锁——轻量级锁失败后，锁膨胀为重量级,此时JVM让线程做几次空循环(即自旋),试图再次获取锁,如果获取成功则进入临界区,如果失败则真正的挂起。

锁消除——JIT编译时，去除不可能出现共享资源竞争的锁。应用场景是在开发时,在不可能出现并发的场景使用了线程安全的api(譬如StringBuffer,Vector之类);这里涉及到逃逸分析——观察一个变量是否会逃出某个作用域,来判断是否能去除内部对该变量(其实就是对象)的加锁操作(譬如作为返回值返回,因而可以被其他线程访问);逃逸分析必须在sever模式下进行——启动时添加-server开启服务器模式,然后配置JVM参数:-XX:+DoEscapeAnalysis=true/false(逃逸分析开关),-XX:+EliminateLocks=true/false(锁消除开关)

### ThreaLocal 保存只有当前线程能访问的资源
但是,如果每个线程分配到的都是同一个对象,依然会有线程安全。所以,ThreaLocal适合可以为每一个线程分配独立副本资源的场景下使用(譬如同一个实例内的成员属性需要线程隔离的情况——自己推测的,未证实!)

其本质是每个线程实例内部持有一个ThreadLocal.ThreadLocalMap(这里有两种一个是threadLocals,一个是inheritableThreadLocals,后者可以获得继承效果获取父类的数据)的Map对象,以ThreadLocal实例引用作为key,实例的值作为value!
### 这里之前理解有误！

ThreadLocal中的ThreadLocalMap是一个类似于WeakHashMap的存在,内部的ThreadLoacal引用(即key)都是弱引用,由于弱引用的机制,当外部ThreadLoacal引用(强引用)回收时,内部的key就变成null了,所以这些线程持有的对象就都被回收了。

使用ThreadLocal之后,如果还有线程池的话,处理不当可能会有内存泄漏的情况,因为对象都是保存在线程中的,线程不退出,对象不会被回收。

### 无锁 CAS compare and swap
依赖现代处理器支持原子化的CAS指令来实现,使用字段偏移量在内存中访问到对应字段,在根据期望值和新值来修改;(依赖Unsafe类的本地方法compareAndSwapXXX,使用指针操作);

Unsafe类无法在有classloader情况下使用,而rt.jar中的类由Bootstarp类加载器加载,Bootstarp不是java类,不需要加载器,因此也没有classLoader的java对象,所以当一个类的类加载为null时,说明是由Bootstrap加载,极有可能是rt.jar中的类,所以可以使用Unsafe类。(这有可能是类加载机制使用双亲委派机制的原因,优先让类库中的类由更高层的加载器来加载)

### CAS逻辑上的不足
比较是数据被修改了2次,改回了原值,则无法判断是否被修改过。

特别是AtomicReference无锁对象引用的限制,是否能修改对象,不完全取决于当前值。

因此,应该使用AtomicStampedReference,带时间戳的无锁对象引用,注意value和timestamp都需要自己更新

AtomicIntegerArray,数组也能无锁,针对具体下标的元素的CAS操作

AtomicXXXFieldUpater,XXX分别代表Integer、Long、Reference,针对普通变量的无锁工具类,只能修改可见范围内变量,变量必须是volatile,不支持static字段,但是暂时不是很清楚应用场景上跟其他的原子类有什么差别,