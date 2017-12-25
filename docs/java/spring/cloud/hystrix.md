### Hystrix：

Hystrix是提供熔断机制的组件，Feign集成了Hystrix；通过spring-cloud与eureka集成后，需要手动开起Hystrix(Dalston.SR4版本默认是关闭的，在配置中使用feign.hystrix.enabled=true可以开启)。

### Hystrix隔离策略：
execution.isolation.strategy设置为
SEMAPHORE则使用信号量来隔离以来的服务,而不使用线程隔离。