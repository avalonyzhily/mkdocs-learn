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