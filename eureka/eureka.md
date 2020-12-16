eureka 服务中心，又称注册中心，主要管理各个服务功能包括服务注册，发现，熔断，降级等。

   eureka基本架构：1 eureka server  提供服务注册和发现

​                                    2 eureka Provider 服务提供方，将自身注册到eureka，从而使服务消费方可以找到

​									3 Service Consumer 服务消费方，从Eureka获取注册服务列表，从而能够消费服务

maven 

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

启动注解 @EnableEurekaServer

yml配置：默认情况下，该服务会注册自己

`eureka.client.register-with-eureka` ：表示是否将自己注册到Eureka Server，默认为true。

`eureka.client.fetch-registry` ：表示是否从Eureka Server获取注册信息，默认为true。

`eureka.client.serviceUrl.defaultZone` ：设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔

集群

eureka使用互相注册的方式实现高可用的部署，只需要将eureka server配置其他可用的serviceUrl

服务提供和调用

   服务提供者

maven

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

启动注解 @EnableEurekaClient

