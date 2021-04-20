# Spring Cloud

服务注册中心：

1. Eureka（×）
2. Zookeeper
3. Consul
4. Nacos（√）

服务调用1：

1. Ribbon（√）
2. LoadBalancer（√）

服务调用2：

1. Feign（×）
2. OpenFeign（√）

服务降级：

1. Hystrix（×）
2. resilience4j
3. sentienl（√）

服务网关：

1. Zuul（×）
2. Zuul2（△）
3. gateway（√）

服务配置：

1. Config（×）
2. Nacos（√）

服务总线：

1. Bus（×）
2. Nacos（√）



原核心组件：

- Eureka：各个服务启动时，Eureka Client都会将服务注册到Eureka Server，并且Eureka Client还可以反过来从Eureka Server拉取注册表，从而知道其他服务在哪里 
- Ribbon：服务间发起请求的时候，基于Ribbon做负载均衡，从一个服务的多台机器中选择一台
- Feign：基于Feign的动态代理机制，根据注解和选择的机器，拼接请求URL地址，发起请求
- Hystrix：发起请求是通过Hystrix的线程池来走的，不同的服务走不同的线程池，实现了不同服务调用的隔离，避免了服务雪崩的问题
- Zuul：如果前端、移动端要调用后端系统，统一从Zuul网关进入，由Zuul网关转发请求给对应的服务
- Config：配置中心
- Bus：消息总线



现在常用阿里巴巴那一套：

- Nacos：致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
- LoadBalancer：负载均衡
- Sentinel：面向分布式服务架构的流量控制组件，主要以流量为切入点，从流量控制、熔断降级、系统自适应保护等多个维度来帮助您保障微服务的稳定性
- Seata：是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

