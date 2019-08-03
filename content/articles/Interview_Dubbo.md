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


## 角色
1. Provider: 暴露服务的服务提供方。
2. Consumer: 调用远程服务的服务消费方。
3. Registry: 服务注册与发现的注册中心。
4. Monitor: 统计服务的调用次调和调用时间的监控中心。
5. Container: 服务运行容器。

## 流程
1. 服务容器负责启动，加载，运行服务提供者
2. 服务提供者在启动时，向注册中心注册自己所要提供的服务
3. 服务消费者在启动时，向注册中心订阅自己所需要的服务——注册中心将服务提供者的路由列表发送给消费者
4. 注册中心基于长链接，从服务提供者处获得服务更新，将服务变动推送给消费者
5. 消费者从路由列表中，基于软负载均衡算法，选一台提供者进行调用，如果失败，尝试使用另一台（哈希一致性算法，随机负载算法，最小连接数，随机）

## 层级结构
1. 服务接口层（service）：与实际业务相关，将服务的实现通过接口暴露出来。
2. 配置层（config）：对外配置接口，以serviceConfig和RefrenceConfig为中心，可以直接new处配置类，也可以通过spring解析配置生成配置类
3. 服务代理层（proxy）：服务接口透明代理，生成服务的客户端Stub和服务端Skeleton，以serviceProxy为中心，扩展接口为ProxyFactory
4. 服务注册层（Registry）：封装服务地址的注册于发现，以服务URL为中心，扩展接口为RegistryFactory、Registry和RegistryService；也可以没有注册中心，服务方直接暴露服务给消费者
5. 集群层（Cluster）：封装多个提供者的路由及负载均衡，并桥接注册中心，以Invoker为中心，扩展接口为Cluster、Directory、Router和LoadBalance。将多个服务提供方组合为一个，使得消费者能够透明的调用服务，根据负载均衡策略选取一个具体Invoker为消费者提供服务
6. 监控层（Monitor）：RPC调用次数和调用时间监控，以Statistics为中心，扩展接口为MonitorFactory、Monitor和MonitorService。
7. 远程调用层（Protocol）：封装RPC调用，以Invocation和Result为中心，扩展接口为Protocol、Invoker和Exporter。Invoker是对象的实体，是核心模型，其他模型也将转换成它，是一个可执行体，可以是本地、集群、远程的方法调用。
8. 信息交换层（Exchange）：封装请求响应模式，同步转异步，以Request和Response为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient、ExchangeServer
9. 网络传输层（Transport）：抽象mina和netty为统一接口，以Message为中心，扩展接口为Channel，Transporter，Client，Server和codec
10. 序列化层（Serialize）：可复用的一些工具，扩展接口为Serialization、ObjectInput、ObjectOutput、TreadPool
 