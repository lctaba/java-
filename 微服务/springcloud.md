## 为什么要有微服务

因为all in one 如果需要容灾，那么需要备份多个单体应用到不同的服务器上。

但是每个应用中总是某些模块调用的多，某些模块调用的少。

于是将他们拆成一个一个的微服务，每个服务都可以独立部署，服务之间通过轻量级的通讯机制进行通讯。



## 引出微服务的四个问题

这么多微服务，客户端怎么调用？  ——网关

这么多微服务，如何进行治理？ ——eureka，服务的注册与发现

服务之间是如何通信的？ ——dubbo，feign

服务挂了怎么办？ ——熔断机制、服务降级



## 微服务的优势

可以将功能解耦，某个微服务出错不影响整个项目，可以灵活地进行版本发布

服务足够内聚，足够小，代码容易理解，能聚焦于某一个业务。

能被小团队开发

容易与第三方集成

有自动部署的工具

不用写前端了

每个微服务都有自己的存储能力



## 缺点

开发人员要处理分布式系统的复杂性

运维难度增加

系统部署依赖复杂

服务间的通讯成本

数据一致性

集成测试困难

性能监控困难







## 集群

将几个提供相同功能的微服务实例同时部署，只是端口不同，那么就可以通过负载均衡策略提高性能，挂了一个其他两个也可以维持使用。

## 服务注册

eureka server将所有微服务注册以便调用

eureka server表明他是注册中心，用注解@EnableEurekaServer

defaultZone：http://${eureka.instance.hostname}:${server.port}/eureka/ 作为服务器，后续微服务注册到注册中心需要这个地址

spring-cloud-starter-netflix-eureka-client 表示这是个 eureka 客户端。

@EnableEurekaClient表明是一个非注册中心微服务

@EnableDiscoveryClient， 表示用于发现eureka 注册中心的微服务。

## Feign

- @FeignClient（value=微服务名）
- @EnableFeignClients表示启用feign方式

## configServer

@EnableConfigServer 表示是配置服务器

配置客户端需要在bootstrap.yml 里配置： serviceId: config-server

## hystrix

注解由原来的
@FeignClient(value = "PRODUCT-DATA-SERVICE")
修改为
@FeignClient(value = "PRODUCT-DATA-SERVICE",fallback = ProductClientFeignHystrix.class)

这就表示，如果访问的 PRODUCT-DATA-SERVICE 不可用的话，就调用 ProductClientFeignHystrix 来进行反馈信息。

## 网关

@EnableZuulProxy开启网关

可以直接用gateway在配置文件配置

网关中还可以用来过滤，但是这边没用









## 问题：

什么是微服务？

微服务之间是如何独立通讯的？

springCloud和Dubbo有什么区别？

谈谈对springBoot和springCloud的理解

什么是服务熔断？什么时候服务降级？

微服务的优缺点是什么？说一下在项目开发中遇到的坑。

- 本地环境配置麻烦，负责的服务相互依赖，本地调试困难
- 分布式事务——最总一致性
- 问题定位很麻烦，需要用zipkin链路追踪
- 日志在不同机器上，看起来很麻烦——ELK日志分析
- 接口稍微改动可能造成其他服务不能运行——契约测试
- 手动部署麻烦：k8s

你所知的微服务技术栈？

eureka和zookeeper的区别？