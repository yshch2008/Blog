中间件，拿一个熟悉的说清楚大致原理，比如kafka的处理过程，，
## kafka
### 概述
1. 作为集群运行在一个或多个服务器上
2. 存储消息以topic为类别记录
3. record由一个k-v-timestamp 构成
4. TCP协议

### 角色
#### 1. Topic
将消息按照topic区分，每个topic相当于一个channel

#### 2.Producer
发布消息的对象

#### 3.Consumer
订阅并处理获取到的消息

#### 4.Broker——代理
集群中的一台服务器，称为代理

### 核心API
1. Producer API ：发布消息到一个或多个topic
2. Consumer API： 订阅一个或多个topic，并处理获取的消息
3. Streams API：充当一个流处理器，从一个或多个topic消费输入流，并产生一个输出流到1个或多个输出topic
4. Connector API：将topic连接到其他程序或数据系统

### 消息如何实现去重
版本号

### 消息如何实现顺序性
1. 全局只有一个生产者
2. 全局只有一个消费者

 Kafka 不支持分布式事务
 丢消息：内存的数据没有写会到磁盘
