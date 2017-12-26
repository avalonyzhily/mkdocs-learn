### eager-load:
这是ribbon的加速第一次调用服务的配置参数,由于zuul也是使用ribbon来实现服务调用的负载均衡,因此也可以使用该配置。但是开启这个配置需要指明具体适用哪些服务，如果使用默认路由,则不需要该配置。

### zuul内置hystrix：
zuul内置hystrix用于服务熔断和降级,因此需要注意hystrix的超时时间,以免出现首次调用超时的问题。

### 两个处理请求的Servlet：
1. DispatcherServlet和ZuulServlet的请求主要通过ServletDetectionFilter这个过滤器来进行区分,检测结果封装到context的isDispatcherServletRequest这个参数中去,可以通过RequestUtils.isDispatcherServletRequest()
和RequestUtils.isZuulServletRequest()方法来判断请求处理的源头
2. DispatcherServlet：主要负责处理由外部发起的http请求(除了/zuul/*这个路径)
3. ZuulServlet：主要处理/zuul/*路径的请求,主要用于处理大文件上传的情况,该路径可以使用zuul.servletPath参数来进行修改

### zuul的忽略服务配置:
```zuul.ignoredServices``` 可以用来忽略不想路由的服务,可以使用*来做通配符,也可以指定具体的服务名```'XXX' or 'xxx*'```,只能是一个字符串,多个可以用逗号分隔```'XXX,YYY,ZZZ*'```