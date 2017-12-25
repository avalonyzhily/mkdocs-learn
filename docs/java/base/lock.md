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