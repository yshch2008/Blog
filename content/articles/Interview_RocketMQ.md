
### 开源项目的理解，底层设计原理（dubbo，rocketmq，oceanDB）

## Rocket MQ
1. 作用：削峰，解耦，最终一致性
2. 

### 总体处理模式：
生产者将消息推送给每个Topic
Topic将消息推送给订阅的Groups
每个Group 根据负载均衡策略选取组内一个Consumer处理该消息

#### Group -consumer
Group内的Consumer可以是多台机器，也可以是多个进程，也可以是一个进程内的多个consumer对象
默认以均摊的方式消费信息，如果是广播模式则每个消息都推给每个consumer

#### Message Queue：
一个Topic下可以由多个Queue
该机制使得消息存储可以分布式集群化，具有水平扩展的能力

#### 顺序队列：
用户实现MessageQueueSelector为某一批消息指定Queue，则这一批消息只在一个queue中，如果消费时只有一个consumer（只有一个Group订阅，group里只有一个consumer），则这一批消息是顺序消费的

#### 事务消息
这样的消息具备多个状态
发送分两个阶段：
1. 发送 prepared状态的消息（这种状态consumer看不到），然后回调用户的TransactionExecutor接口，开启事务操作
2. 事务操作成功时，提交该消息，这是对consumer是可见的
这就保证了消息提交的事务性（失败了consumer看不到，所以不消费）

#### 存储
每个CommitLog对应一个ConsumeQueue,每个ConsumeQueue对应一个MessageQueue
消息主体经过CommitLog读写，ConsumeQueue中值储存少量数据

#### Broker 主从结构
通过制定相同的BrokerName，不同的BrokerID来定义，BrokerID=0则为master，否则是slave

#### Broker-producer
Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从 Name Server 取 Topic 路由信息，并向提供 Topic 服务的 Master 建立长连接，且定时向 Master 发送心跳。

#### Broker-consumer
Consumer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向 Master、Slave发送心跳。

#### consumer消费模式
1. 集群消费：一个topic可以由同一个ID下所有消费者分担消费。
2. 广播消费：每个消费者消费Topic下的所有队列。