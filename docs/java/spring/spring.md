### Spring Bean的生命周期简述

1. BeanFactoryPostProcessor接口的postProcessBeanFactory->
2. InstantiationAwareBeanPostProcessor接口的postProcessBeforeInitializations方法调用->
3. Bean构造调用->
4. 属性注入->
5. 实现aware接口的方法执行->
6. BeanPostProcessor接口的postProcessBeforeInitializations方法->
7. 注解的初始化方法(@PostConstruct注解扫描生效)->
8. 实现initializingBean接口的afterPropertiesSet()方法->
9. bean标签init-method属性配置的方法或beans标签default-init-method属性配置的方法(xml配置生效)->
10. BeanPostProcessor的postProcessAfterInitialization方法->
11.  InstantiationAwareBeanPostProcessor接口的postProcessAfterInitialization方法调用->
12.  Bean初始化完成->
13.  Bean的销毁->
14.  注解的销毁前置方法(@PreDestroy注解扫描生效)->
15.  实现DisposableBean接口的destroy()方法->
16.  bean标签destroy-method属性配置的方法或beans标签default-destroy-method属性配置的方法(xml配置生效). 
17.  在Bean初始化完成之后,会去根据应用的状态去执行实现了Lifecycle(及其扩展接口如SmartLifecycle等)里面对应的方法(start(),stop()等等)。

### 关于BeanPostProcessor的一些应用注意点

1. BeanPostProcessor接口的实现类会在Spring初始化Bean时作用于每一个非BeanPostProcessor及相关Bean,插入自定义的操作，里面有前置和后置两个方法需要实现，分别在Bean的初始化方法(@PostConstruct)之前和(init-method)之后执行，方法中可以获取到bean对象和beanName;
2. BeanPostProcessor的实现与普通的bean一样在Spring容器中进行加载，但是不会处理其本身以及其他的BeanPostProcessor,譬如AOP的自动代理(auto-proxy)就是基于BeanPostProcessor实现的,因此BeanPostProcessor是不能被代理的;
3. BeanPostProcessor的实现中不要注入别的bean,因为此时Bean的初始化并没有完成,可能会发生你意想不到的情况.

### 关于Spring处理配置文件的工具类之一:PropertyPlaceholderConfigurer

1. PropertyPlaceholderConfigurer读取的配置可通过占位符的形式编入Spring的配置xml中(${XXX},spel规范),应用运行时替换;
2. PropertyPlaceholderConfigurer:xml标签写法<context:property-placeholder location="classpath:jdbc.properties"/>等价于<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:jdbc.properties"/>
</bean>;
3. PropertyPlaceholderConfigurer也会去检测系统变量中的配置,通过systemPropertiesMode属性的三个值来制定检测行为：默认时1(fallback),即优先使用制定配置文件的配置,没有就用系统的;0(never),只用指定配置文件的,不用系统的;2(override),优先使用系统配置覆盖指定配置文件中的配置。

### 关于Spring处理配置文件的工具类之二:PropertyOverrideConfigurer

1.PropertyOverrideConfigurer主要用于读取配置文件之后,替换原有的配置值,自上而下的读取配置,后读取的同名配置覆盖前面的配置。


### Spring的自动装载@Autowired注解

1. 当有多个bean匹配的时候,可以通过@Qualifier注解进行匹配的限定,还可以使用@Primary来确定想要注入的bean;
2. @Qualifier用于匹配bean定义中设置的qualifier值(xml中为bean标签内嵌的qualifier标签),无法匹配时则匹配字段名;
3. @Primary用在bean的定义处(或xml中bean标签的primary属性)
4. <qualifier/>标签内嵌<attribute key="" value=""/>标签也可以用<meta key="" value=""/>来替换,但是优先使用前者。
5. @Autowired还可以通过泛型进行匹配(注意一下)
6. 截至Spring4.3,@Autowired可以使用自引用(即bean内部注入自己本身的bean),这种方法是非正规的,但是在实践中,如果遇到同一个类里面某个方法调用其它带有AOP性质的方法时会失效的情况,使用自引用则可以避免失效。


### Spring使用JSR330标准注解需要额外的以来：

以maven为例：
```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

### 启动Web应用可以不用默认的XmlWebApplicationContext:

也可以使用AnnotationConfigWebApplicationContext,web.xml配置如下

```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

### 关于注解@PropertySource：

注解```@PropertySource```可以加载配置文件,加载的配置对应```Spring```中的```Environment```类,
同时```@PropertySource```的元数据中接受```${...}```的表达式,例如:```${my.placeholder:default/path}```其中```my.placholder```为已存在的配置资源变量,如果不存在则使用默认值```default/path```,没有默认值的话,找不到对应变量则会抛出异常。

### TaskExecutor  Executor的Spring抽象和封装
TaskExecutor 接口集成了java的并发包中的Executor接口,是Spring对Java中线程池使用需求的一个抽象;原本是提供给Spring的一些组件使用的。

TaskExecutor 有很多类型,几乎覆盖了所有的使用需求：

1. SimpleAsyncTaskExecutor 简单的异步执行器,不会复用线程,每次都会新开启一个线程去执行任务;

2. SyncTaskExecutor 同步执行器,不会异步执行,直接在调用线程内执行每一个任务。

3. ConcurrentTaskExecutor 并发执行器,Executor的一个适配器,将Executor的配置参数作为一个bean的配置来暴露出来,针对ThreadPoolTaskExecutor不够灵活时,可以使用该执行器。

4. SimpleThreadPoolTaskExecutor 是Quartz的SimpleThreadPool的子类,常用场景是在Quartz的组件和非Quartz的组件之间共享线程池。

5. ThreadPoolTaskExecutor 最常用的,对外暴露一个bean来配置ThreadPoolExecutor,然后封装成一个TaskExecutor(ThreadPoolTaskExecutor的父类);如果要适配不同类型的Executor,还是推荐使用ConcurrentTaskExecutor;

6. WorkManagerTaskExecutor CommonJ(计时器和工作管理器)是BEA和IBM的标准,不是JavaEE的标准。类似SimpleThreadPoolTaskExecutor

7. 简单的例子
```
import org.springframework.core.task.TaskExecutor;

public class TaskExecutorExample {

    private class MessagePrinterTask implements Runnable {

        private String message;

        public MessagePrinterTask(String message) {
            this.message = message;
        }

        public void run() {
            System.out.println(message);
        }

    }

    private TaskExecutor taskExecutor;

    public TaskExecutorExample(TaskExecutor taskExecutor) {
        this.taskExecutor = taskExecutor;
    }

    public void printMessages() {
        for(int i = 0; i < 25; i++) {
            taskExecutor.execute(new MessagePrinterTask("Message" + i));
        }
    }

}

//xml config
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
    <property name="corePoolSize" value="5" />
    <property name="maxPoolSize" value="10" />
    <property name="queueCapacity" value="25" />
</bean>

<bean id="taskExecutorExample" class="TaskExecutorExample">
    <constructor-arg ref="taskExecutor" />
</bean>
```

### TaskScheduler 定时任务的Spring抽象和封装
TaskScheduler 是一个定时任务的接口定义,同时还需要触发器来搭配使用(即定义任务的执行时间和周期等配置),可以使用CronTrigger和PeriodicTrigger两种触发器实现,这是Spring提供的。

TaskScheduler其实就是TaskExecutor的周期性行为,所以是依赖TaskScheduler的代码实现的。

1. ConcurrentTaskScheduler 和 DefaultManagedTaskScheduler 都继承自ConcurrentTaskExecutor

2. ThreadPoolTaskScheduler 是ThreadPoolTaskExecutor的周期性行为

### 注解的形式启动定时任务
1. ```@EnableScheduling``` 作为配置类的注解,用于激活定时任务配置;

2. ```@Scheduled``` 用来注解方法,表明该方法是一个周期性执行的任务;其中fixedDelay表示每次执行完后延迟X毫秒,再执行;fixedRate表示每隔X毫秒,执行;initialDelay表示初始延迟X毫秒;也可以用cron表达式来定义执行周期(此方法更具有操作性和灵活性)

3. spring4.3之后,```@Scheduled``` 支持任意域的bean;这里要注意一般不要用多实例域的bean来使用该注解,否则会启动多个定时任务,除非你确定需要这样;另外也不要在```@Configurable```注解的类中使用,否则会出现2个容器,导致定时任务执行2次

### 注解形式激活异步处理
1. ```@EnableAsync``` 作为配置类的注解,用于启用异步处理(其实就是配置线程池信息)
```
@Configuration
@EnableAsync
public class AsyncConfig {
    /** Set the ThreadPoolExecutor's core pool size. */
    private int corePoolSize = 10;
    /** Set the ThreadPoolExecutor's maximum pool size. */
    private int maxPoolSize = 200;
    /** Set the capacity for the ThreadPoolExecutor's BlockingQueue. */
    private int queueCapacity = 10;

    private String ThreadNamePrefix = "ParallelDealDataInfoService-Executor-";

    @Bean
    public Executor dealDataInfoExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setThreadNamePrefix(ThreadNamePrefix);

        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

2. ```@Async``` 注解在方法上标明该方法是异步方法,一般来说没有返回值,但是可以返回Future/ListenableFuture、CompletableFuture(需要spring4.2 java8)对象来获取返回值(具体原来是基于java本身的多线程工具)

3. ```@Async```不能用于生命周期中的回调方法,比如```@PostConstruct```,只能在其中调用一个异步方法;意外本类中调用异步方法是无法触发异步效果的(因为AOP的原因,同一个类,没有注解的方法调用有注解的方法,代理中也不会加入异步的代码而直接调用原来的方法)

4. ```@Async("otherExecutor")``` 这个配置就是指定一个Executor的bean,即自定义执行器配置,默认是用的```SimpleAsyncTaskExecutor```这个没有复用线程,没有控制线程数,会有性能问题

5. 有Future返回的异步方法,管理异常很简单,在Future对象调用get()方法时可以拿到异常信息;但是没有返回的异步方法,则需要定义相应的AsyncUncaughtExceptionHandler来处理(可以通过```AsyncConfigurer ```或XML标签)
