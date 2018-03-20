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