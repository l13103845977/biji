**网关的作用**

统一入口:为所有服务提供一个统一的入口

健全校验：识别每个请求的权限

动态路由：动态的将请求路由到不同的后端集群中

减少客户端和服务端的耦合：服务可以独立发展，通过网关层来做映射

**通过网关访问方式**

默认地址为: http://zuulHostIp:port/要访问的服务名称/服务对外提供的url路径监听

maven仓库

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

启动注解@EnableZuulProxy

yml配置

 可以使用的通配符有 * ** ？

path 使用路径方式匹配路由规则，参数key结构 zuul.routs.服务名称.path=xxx 

​     url用于配置符合path的请求路径路由到的服务地址：zuul.routs.服务名称.url=127.0.0.1:8080

​     serviceId 用于服务名称匹配符合path的请求路径路由到的服务地址 zuul.routs.服务名称.serviceId =***

​     路由排除配置

​             1 ignored-services用于配置不被zuul代理的服务，多个服务用逗号隔开，如果值为*，则相当于给所有新发现的服务排除zuul网关访问方式，只有配置了路由网关才能用

​              2     ignored-patterns 用于通配方式排除代理路径，所有符合ignored-patterns的请求路径都不被zuul代理

​         路由前缀配置 prefix



**zuul网关过滤器**

​     zuul提供了过滤器父类ZuulFilter，有4个方法 ，

  1.通过父类中的抽象方法filterType，决定当前是什么filter

- **前置过滤**（pre）：是请求进入Zuul之后，立刻执行的过滤逻辑。
- **路由后过滤**（route）：是请求进入Zuul之后，并Zuul实现了请求路由后执行的过滤逻辑，路由后过滤，是在远程服务调用之前过滤的逻辑。
- **后置过滤**（post）：远程服务调用结束后执行的过滤逻辑。
- **异常过滤（error）**：是任意一个过滤器发生异常或远程服务调用无结果反馈的时候执行的过滤逻辑。无结果反馈，就是远程服务调用超时。

2.filterOrder 调用filter顺序，数字越小，顺序越靠前

3.shouldFilter Boolean类型，当前filter是否生效

4.run，具体执行的逻辑，该方法可以返回任何对象，返回null即可，通过zuul获取请求上下文，zuul实现的RequestContext类

 过滤器的生命周期

![](E:\biji\springcoudstudy\网关\1010726-20191018040056721-2124170892.png)

**zuul网关的容错**

zuul的网关容错只针对网络超时，新版本提供了fallbackprovider接口实现，共有两个方法

  1.getRoute返回当前的容错逻辑处理的是哪一个服务（服务名称），可以使用通配符*或null代表所有

2. fallbackResponse返回ClientHttpResponse，网关向api的请求失败了，但是客户向网关发起的请求是ok的，所以不能返回404，500等错误。 

    返回一个匿名对象new ClientHttpResponse{}重写该对象的方法。一般设置状态码为ok，主要实现方法getbody和getheaders，getbody里返回的内容，getheaders返回编码，和getbody中的编码保持一致。

 

zuul网关的限流保护

maven 

```
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>1.3.4.RELEASE</version>
</dependency>
```

yml配置

局部限流

```javascript
zuul.ratelimit.policies.服务名称.limit=3
zuul.ratelimit.policies.服务名称.refresh-interval=60
##针对某个 IP 进行限流，不影响其他 IP
zuul.ratelimit.policies.服务名称.type=origin
```

 全局限流

```
zuul:
  ratelimit:
    enabled: true
    default-policy:
      limit: 3
      refresh-interval: 60
      type: origin
```

![](E:\biji\springcoudstudy\网关\1010726-20191018043154762-1560322806.png)

异常处理 ，新增一个异常处理类，实现errorController接口，实现getErrorPath方法，返回一个指向错误资源的url

    ratelimit:
        key-prefix: your-prefix  #对应用来标识请求的key的前缀
        enabled: true
        repository: REDIS  #对应存储类型（用来存储统计信息）
        behind-proxy: true  #代理之后
        default-policy: #可选 - 针对所有的路由配置的策略，除非特别配置了policies
          limit: 10 #可选 - 每个刷新时间窗口对应的请求数量限制
          quota: 1000 #可选-  每个刷新时间窗口对应的请求时间限制（秒）
          refresh-interval: 60 # 刷新时间窗口的时间，默认值 (秒)
          type: #可选 限流方式
            - user
            - origin
            - url
          policies:
            myServiceId: #特定的路由，服务名称
              limit: 10 #可选- 每个刷新时间窗口对应的请求数量限制
              quota: 1000 #可选-  每个刷新时间窗口对应的请求时间限制（秒）
              refresh-interval: 60 # 刷新时间窗口的时间，默认值 (秒)
              type: #可选 限流方式
                - user
                - origin
                - url
zuul调优

zuul底层采用了ribbon和hystriy，

