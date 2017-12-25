### 静态资源配置问题

spring.resources.static-locations——这个配置是通用的静态资源目录配置,默认的起始路径是classpath:xxxx,这里即使在jar包之外也是支持的,非springmvc的情况下,也必须放在jar包之外的平级目录(也就是类路径的根目录);

spring.mvn.static-path-pattern——这个配置是mvc专有的,默认是/**,也就是默认处理所有的静态资源路径,跟前面不同的是可以支持jar包内部的静态资源访问。(应该是不同的ResourcesHandler)