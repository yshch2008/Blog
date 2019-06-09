### 6.Spring事务的原理，以及嵌套事务怎么处理。

`原理： TransactionSynchronizationManager通过ThreadLocal 给每个线程绑定一个事务，这样在线程请求事务的时候，TSM就会去查询当前线程绑定的事务。通过事务传播策略执行对应操作。`

1. Spring的事务是通过AOP代理类中的一个Advice（TransactionInterceptor）进行生效的，而传播级别定义了事务与子事务获取连接、事务提交、回滚的具体方式。

1. TxAdviceBeanDefinitionParser => 核心实现类TransactionInterceptor
2. TransactionSynchronizationManager 事务同步管理的核心类，它实现了事务同步管理的职能，包括记录当前连接持有connection holder，保证当前激活的连接只能是一个

1. PROPAGATION_REQUIRED -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 
2. PROPAGATION_REQUIRES_NEW -- 新建事务，如果当前存在事务，把当前事务挂起。
3. PROPAGATION_NESTED -- 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。  
### 嵌套事务：
PROPAGATION_REQUIRES_NEW ：完全是一个新的事务
PROPAGATION_NESTED 则是外部事务的子事务, 如果外部事务 commit, 潜套事务也会被 commit, 这个规则同样适用于 roll back。
`内部的撤回不会对外部造成影响，而外部的操作决定内部操作`