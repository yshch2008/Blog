## dubbo rpc框架的基本结构
## 什么是服务治理
服务庞大之后，需要按照科学规范的方法对服务进行治理，包括线下的文档 ，流程，权限管理，线上的服务调度，还有保障服务质量，运维能力等。

服务质量：故障管理，报警系统，性能表现，链路监控，依赖监控
线下治理：开发工具，上限流程，下线通知，服务文档
线上治理：服务发现，服务调度（升降级熔断，监控，负载均衡），服务路由，配置管理
运维生态：用户角色管理流程，服务的流程审批

## 服务注册
spring解析扩展标签
实现NamespaceHandler类
每个service方法的配置会解析成对应的serviceBean

ServiceConfig.export()
 ServiceConfig.doExport() -> ServiceConfig.doExportUrls() -> ServiceConfig.doExportUrlsFor1Protocol()
 RegistryProtocol.export
最终会走到DubboProtocol.export方法通过netty开启服务
openServer方法会根据key（ip+port）判断server是否存在，不存在则调用createServer(URL url)方法创建一个NettyServer来开启服务,
最终会调用ZookeeperRegistry.doRegister注册服务

## 服务发现的过程和原理
1、服务提供者在暴露服务时，会向注册中心注册自己，具体就是在${service interface}/providers目录下添加 一个节点（临时），服务提供者需要与注册中心保持长连接，一旦连接断掉（重试连接）会话信息失效后，注册中心会认为该服务提供者不可用（提供者节点会被删除）。
2、消费者在启动时，首先也会向注册中心注册自己，具体在${interface interface}/consumers目录下创建一个节点。
3、消费者订阅${service interface}/ [ providers、configurators、routers ]三个目录，这些目录下的节点删除、新增事件都胡通知消费者，根据通知，重构服务调用器(Invoker)。
以上就是Dubbo服务注册与动态发现机制的原理与实现细节。

作者：何何与呵呵呵
链接：https://www.jianshu.com/p/9271068e0caf
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。


消费者订阅subscribeUrl的通知，此过程中会把invoker封装为invokerDelegate并在RegistryDirectory中缓存urlInvokerMap。这就完成了服务的发现。