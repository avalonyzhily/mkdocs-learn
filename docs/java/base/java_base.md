### ScheduledExecutorService:
1. 用作定时任务,相比Timer的单线程执行任务更好;
2. java并发包下的类;

### Java类的复用
1. 组合——显示的在新类中放置子对象，通常适合在新类中使用现有类的功能而不是它的接口。has-a
2. 继承——使用现有类的特殊版本时，使用继承。is-a
3. 代理
4. 创建新类时，应该优先考虑使用组合，一味使用继承会减少设计的灵活性，加重设计负担。

### 多态陷阱
1. 子类的实例化会调用父类的构造器，如果在父类的构造器中调用被子类覆盖的方法，会导致调用到子类的方法，而此时子类的初始化还未完成(子类构造器还没有执行完成)，会出现意想不到的情况。
2. 因此，在构造器中尽量避免调用其他的方法，减少意外的情况发生。
3. 向上转型是安全的，向下转型是不安全的，可能抛出ClassCastException

### 协变返回类型(JAVA5以后支持)
子类中覆盖了父类的方法，其返回类型可以时父类中的方法的返回类型的子类(修改返回类型不改变方法的签名，子类覆盖的方法可以缩小返回类型的范围)，这样在向上造型时，调用该方法不会导致返回类型超出范围的情况。

### interface接口 abstract抽象类
1. 接口是类与类之间的协议，不需要提供具体实现，也不能提供具体实现，接口中只能有常量或非常量的表达式(依然是静态的)。(Java8开始，接口中可以有多个默认方法，即默认实现，间接实现多继承)

2. 接口的实现不受继承的单一限制，一个接口可以继承多个接口，这里要注意多个接口组合时要避免方法签名相同的情况，会出现混淆和错误

3. 一个类可以实现多个接口，但只能继承一个类(包括抽象类)，而且必须先继承一个类，再实现多个接口(这是同时存在继承和实现的时候)

4. 对于抽象类和接口的选择，如果要创建一个没有任何方法实现和成员的父类，应该选择接口；反之，可以选择抽象类。但是一般来说优先使用接口作为父类，来定义行为规范

5. 抽象类则可以包含一部分带实现的方法和一部分不带实现的方法声明。

6. 接口和抽象类都无法实例化。

7. 接口可以嵌套在类或接口中，类中嵌套的接口(或类)可以使private的，但这些嵌套接口的实现类都是内部类，只能在类的内部使用。(这里不是很理解意义，用法也没怎么见过)

8. 接口的经典用法——工厂模式(方法)

### adapter适配器模式
接受拥有的类型/接口，实现/继承(单一限制)需要的接口，来生成需要的类型，这里用到了代理
```
public interface Processor{
    String name();
    Object process(Object input);
}

class FilterAdapter implements Processor{
    Filter filter;
    public FilterAdapter(Filter filter){
        this.filter = filter;
    }
    public String name(){
        return filter.name();
    }
    public Object process(Object input){
        return filter.process((Waveform)input);
    }
}
```

### 接口使用的误区
只有真正有抽象需求时，才应该使用接口;否则，应该优先使用类，减少设计的复杂度。接口是重要的工具，但是不应该被滥用。(注意这里与 接口和抽象类的选择 的不同理解)

### 内部类
1. 内部类拥有外部类所有元素的访问权，包括私有的;非静态的内部类的对象，只能在与外部类的对象关联时才能创建(即是外部类的对象持有的)

2. Class.this 外部类对象的引用，Object.new 通过外部类对象实例化内部类

3. 内部类向上转型为其父类，则可以阻止依赖类型的编码，以及隐藏实现细节(私有和受保护的内部类实现模个外部公有的接口，外部可以通过接口获得该内部类的实例，但不能向下转型为该内部类)

### Java8新特性之方法作为参数传递
1. 方法引用(双冒号)——对象::实例方法,类::静态方法,等同于提供方法参数的lambda表达式,如Math::pow 等同于 (x,y)->Math.pow(x,y);类::实例方法,其第一个参数会作为调用方法的对象,如 String::compareToIgnoreCase 等同于(x,y)->x.compareToIgnoreCase(y);还有this::实例方法,super::实例方法(这个是用this作为执行方法的对象,调用的是父类的方法) 

2. 函数式接口——有很多类型(java.util.function包下),用于接收函数或lambda表达式,这里不能将函数视为对象,因为函数并不是继承Object的。

3. lambda表达式可以转换成函数式接口; 方法引用同样也会转换为函数式接口的实例,在重载问题上,会根据对应函数式接口的方法参数来匹配

4. 构造引用——类::new

### 饿汉式和懒汉式(同步)单例的结合
```
/*
私有静态内部类,屏蔽外部的访问,又利用JVM类初始化机制创建单例
*/
public class StaticSingleton{
    private StaticSingleton(){
        //....
    }

    private static class SingletonHolder{
        private static StaticSingleton instance = new StaticSingleton()
    }

    public static StaticSingleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

### 不变模式天生支持多线程,却是通过回避并发问题来实现的，并不是解决
不变模式只是对象创建后就不会改变,不论是外部操作还是对象本身的操作(与只读不同,只读并不控制对象本身的操作)

java中的不变模式实现,finanl修饰类和属性,属性私有,去除setter,拥有创建完整对象的构造,还要确保没有子类可以重载修改方法(final修饰类)。

java中的String和包装类都是不变模式


### 生产者-消费者模式
生产者线程与消费者线程通过共享内存缓存区(譬如:BlockingQueue)来通信,而不直接通信。

```
//使用BlockingQueue作为共享内存缓存区
//BlockingQueue在高并发场景性能不是很优秀,使用无锁的ConcurrentLinkedQueue高并发//场景性能很好,但是缺少Blocking的作用,会瞬间耗尽cpu占用(亲测有效)
public class PCData {
    private final int intData;

    public PCData(int intData) {
        this.intData = intData;
    }

    public PCData(String intData) {
        this.intData = Integer.valueOf(intData);
    }

    public int getIntData() {
        return intData;
    }
}


public class Producer implements Runnable {
    private volatile boolean isRunning = true;
    private BlockingQueue<PCData> queue;
    private AtomicInteger count = new AtomicInteger();
    private static final int SLEEPING = 1000;

    public Producer(BlockingQueue<PCData> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        PCData pcData = null;
        Random random = new Random();
        System.out.println("start producer---id="+Thread.currentThread().getId());
        try {
            while (isRunning){
                Thread.sleep(random.nextInt(SLEEPING));
                pcData = new PCData(count.incrementAndGet());
                System.out.println(pcData+" is put into queue");
                if(!queue.offer(pcData,2, TimeUnit.SECONDS)){
                    System.err.println("fail to put pcData : "+pcData);
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        }
    }

    public void stop(){
        isRunning = false;
    }
}

public class Consumer implements Runnable {
    private BlockingQueue<PCData> queue;
    private static final int SLEEPING = 1000;
    public Consumer(BlockingQueue<PCData> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        System.out.println("start consumer id = "+Thread.currentThread().getId());
        Random random = new Random();
        try{
            while(true){
                PCData pcData = queue.take();
                if(null!=pcData){
                    int re = pcData.getIntData()*pcData.getIntData();
                    System.out.println(MessageFormat.format("{0}*{1}={2}",pcData.getIntData(),pcData.getIntData(),re));
                    Thread.sleep(random.nextInt(SLEEPING));
                }
            }
        }catch (InterruptedException e){
            e.printStackTrace();
            Thread.currentThread().interrupt();
        }
    }
}

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<PCData> queue = new LinkedBlockingQueue<>();
        Producer producer1 = new Producer(queue);
        Producer producer2 = new Producer(queue);
        Producer producer3 = new Producer(queue);

        Consumer consumer1 = new Consumer(queue);
        Consumer consumer2 = new Consumer(queue);
        Consumer consumer3 = new Consumer(queue);

        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(producer1);
        executorService.execute(producer2);
        executorService.execute(producer3);

        executorService.execute(consumer1);
        executorService.execute(consumer2);
        executorService.execute(consumer3);


        Thread.sleep(10*1000);

        producer1.stop();
        producer2.stop();
        producer3.stop();

        Thread.sleep(1000);

        executorService.shutdown();
    }
}
```

### Disruptor 无锁缓存框架
RingBuffer结构用来作为共享缓存区,一开始就定义好可用的数据缓存池(对象实例)的个数(必须是2的幂),因为是环状的,所以可以一直复用,

消费者实现WorkHandler接口,然后实现onEvent方法,在其中处理要完成的任务或要处理的数据(Disruptor封装了读取数据的方法);

生产者持有RingBuffer的实例,向其中插入数据时,首先需要获取下一个序列号,然后根据序列号获取可用的数据缓存池(对象实例),再将数据写入。

使用ByteBuffer来包装任意的数据类型,二进制流可以覆盖所有的数据类型。

Disruptor的写入和读写都使用了CAS算法(内部封装)。