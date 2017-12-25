### Spring cloud ribbon自动配置类:
1. eureka ribbon--- org.springframework.
cloud. netflix. ribbon. eureka. RibbonEurekaAutoConfiguration
2. eureka consul---org.springframework.cloud.consul.discovery. RibbonConsulAuto-
Configuration

### ribbonClient配置优化(C版以上)：
org.springframework.cloud.netflix.ribbon.PropertiesFactory
NFLoadBalancerClassName---配置ILoadBalancer接口的实现;
NFLoadBalancerPingClassName---配置IPing接口的实现;
NFLoadBalancerRuleClassName---配置IRule接口的实现;
NIWSServerListClassName---配置ServerList接口的实现;
NIWSServerListFilterClassName---配置ServerListFilter接口的实现。

ps:对于Ribbon 参数的key以及value类型的定义，可以通过查看com.netflix.client.
config.CommonClientConfigKey类获得更为详细的配置内容

### ribbon的配置说明：
全局配置---ribbon.<key>=<value>
指定客户端---<client>.ribbon.<key>=<value>

ps:其中client就是要调用的服务的id(eureka中)或者ribbonClient声明的名称

### eureka & ribbon: 
ribbon.eureka.enabled可以开启或关闭spring-cloud-eureka对ribbon的控制，关闭则回到原生ribbon的使用;

### 关于ribbon重试机制的问题：
在没有feign的情况下,ribbon重试测试无效,原因未知,待研究。

### ribbon的eager-load:
用于加速第一次调用服务的配置参数，但是开启这个配置需要指明具体适用哪些服务。