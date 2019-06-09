#### AQS原理：
  先进先出（FIFO）等待队列的实现阻塞锁和同步器的框架。
  1. 维护一个队列，其中的Node节点用来放置线程元素，并且包含head,pre,next,tail,state,waitStatus,nextWater等信息，通过对这些信息的更新和判断，实现线程的互斥和协作。
  2. 所有值的更新方式都是CAS
  3. 线程的等待主要是unsafe.park方法，候选节点是自旋方式
#### CAS原理：
  1. 一个值存在两份，一份是期望值，一份是根源值或者叫实际值。
  2. 执行更新操作时分两步，一步判断一步更新，期望与实际相符则更新，否则放弃操作。
  3. 取内存值的操作主要是利用atomicInt类型，调用native的

####线程的状态：
1. NEW,新建，尚未启动
2.    RUNNABLE,正在虚拟机中执行
3.    BLOCKED,阻塞，等待一个monitor锁
4.    WAITING,无限期等待另一个线程的某动作=>Object.wait()   Thread.join()  LockSupport.park()
5.    TIMED_WAITING,限时等待=》Thread.sleep(long) Object.wait(long) Thread.join(long) LockSupport.parkNanos() LockSupport.parkUntil()
6.    TERMINATED;中止状态，已退出

#### interrupt
1. 可以终结自己
2. 在checkAccess()通过后可以调用其他线程的interrupt，runnable状态的，会将其isInterrupted设置为true，线程会以此为依据中断自己
3. 中断处于阻塞状态的（sleep,join wait会抛异常InterruptedException）

#### 可以调用Thread的suspend或者stop中止线程,interrupted() 和 isInterrupted()都能够用于检测对象的“中断标记”。区别是，interrupted()除了返回中断标记之外，它还会清除中断标记(即将中断标记设为false)；而isInterrupted()仅仅返回中断标记

#### sleep wait yield join
1. sleep()方法让线程暂停执行指定的时间，让出CPU给其他线程，但保留已获取的锁，不推荐
2. wait()导致当前线程进入waiting状态，直到请求的对象执行notify/notifyAll操作，释放获取的锁
3. yield()
#### 线程池的工作模式：
  1. corePoolSize没有满，则new线程执行任务（可以使用prestartCoreThread/all提前建满核心线程）
  2. corePoolSize满了，新来的任务会存在阻塞队列，当阻塞队列长度小于corePoolSize时，不新建线程，而是等coreThread空闲时分配
  3. (有界)阻塞队列长度加上corePoolSize应小于等于maxPoolSize，阻塞满了之后会创建新的线程执行任务
  3. （无界）则不存在阻塞队列满的情况，那么只是机械的阻塞新任务，核心线程不断从阻塞队列中取出并执行任务
  4. 线程池中工作的线程数量已经等于maxPoolSize，而阻塞队列又满了，再接到新任务，则执行拒绝策略
  4. 拒绝策略：默认抛异常，忽略请求，替换掉执行时间最长的任务，放入队列等待call

#### Fork/join：
1. 建立工作线程池，用以执行分解出的子线程，父线程需要等待子线程执行完毕才继续执行
2. FJ框架中FJTask虽然也实现了runnable接口，但在FJTask受限规则下运行，共享了父线程的上下文资源，因此线程切换执行时更方便
3. 特殊的调用机制，专用队列，有人物类中的几个方法，主要是join,fork,isDone
4. 适用于递归场景，任务不断分解成子任务，子任务递归完成后得到需求的结果
每个线程维护自己的的任务队列，为双端deques
后进先出的策略执行任务
队列为空则尝试以先进先出的方式从其他线程的队列里取出任务执行（体现出deques的作用）
遇到join操作，则先去执行其他任务，join的操作完成了再继续执行
当线程没有任务可以执行时，将回退yeild\sleep\优先级调整，并稍后再试


#### volatile保证可见性，但不保证原子性，比如它修饰的变量++等依赖自身的操作
volatile修饰数组只保证数组的引用，内部元素享受不到
atomicInteger等是原子操作，cas
synchronized最安全，开销最大，主要靠对象头中的state 实现
threadLocal ，以thread为key，用value弱引用存储变量的副本，一般使用private static修饰（否则该Node存在堆上，key可能会被GC回收掉，而value上存在被之前thread强引用标记，所以不用的时候要手动remove），用static修饰则thread生命周期更长，不会被回收，这样就不会导致既找不到又存在强引用的内存泄漏问题

轻量锁：在栈帧中创建锁记录Lock Record区域，将锁对象头中的mark Word拷贝到锁记录中，再尝试用CAS将Mark Word更新为指向锁记录的指针，不成功则存在竞争，升级为重量级锁
偏向锁：markword里存线程ID，相同线程竞争时直接成功，cas实现

#### 对象=对象头+实例数据+占位符
对象头=Mark Word+类型指针
Mark Word=hashCode ,GC 分代年龄，锁状态标记，线程持有的锁，偏向线程的ID，偏向的时间戳
锁状态标记为 栈中锁记录的指针（轻量级锁），互斥量（重量级锁）的指针， 无锁时存hashCode，偏向锁存线程的ID

hashMap null的key存在table[0] 的第一个位置

CurrentHashMap的初始化一共有三个参数，一个initialCapacity，表示初始的容量，一个loadFactor，表示负载参数，最后一个是concurrentLevel，代表ConcurrentHashMap内部的Segment的数量，ConcurrentLevel一经指定，不可改变，后续如果ConcurrentHashMap的元素数量增加导致ConrruentHashMap需要扩容，ConcurrentHashMap不会增加Segment的数量，而只会增加Segment中链表数组的容量大小，这样的好处是扩容过程不需要对整个ConcurrentHashMap做rehash，而只需要对Segment里面的元素做一次rehash就可以了。

#### 如何创建http请求
httpClient int statusCode = httpClient.executeMethod(getMethod);
httpURLConnection
原生httpClient  response = httpClient.execute(httpGet);

#### 类加载器
根据指定全限定名称将class文件加载到JVM内存，转为Class对象
启动类java_home/bin目录 或Xbootclasspath路径中的类库
扩展类Java_home/bin/ext或java.ext.dir系统变量指定的路径中的所有类库
应用程序加载用户路径classpath上的指定类库，可以直接使用

#### 双亲委派模型
如果一个类加载器收到类加载的请求，他不会自己尝试去直接加载这个类，而是把这个请求委派给父类加载器完成。
只有在父类记载其在iji的搜索范围内找不到指定的类时(classNotFoundException)

目的： 默认类库有的，会先用默认类库，安全，防止恶意加入混淆的类
确保一个类的全局唯一性
打破双亲委派机制不仅要继承ClassLoader类，还要重写LoadClass和findClass方法

#### RPC优缺点
RPC，可以基于TCP协议，也可以基于HTTP协议
使用自定义的TCP协议，可以让请求报文体积更小
可以基于二进制传输，编解码复杂度小
基本自带负载均衡
能做到自动通知调用者，而不需要额外通信

伪装成普通方法调用，编程时更加专注于代码逻辑而不需要处理网络封装和拆箱
异步的，因此不需要考虑结果什么时候返回

缺点：多个服务叠加可能维持的线程总数很多


#### ZGC
三个非常短暂的STW的阶段——根集合（root set）扫描
动态地决定Region的大小
从实例指针中借位表示finalizable，remapped marked1，marked2
整理操作——而非清理和复制
没有G1的Remember Set 没有write barrier开销
当一条线程分配对象时，根据当前是哪个CPU在运行的，就在靠近这个CPU的内存中分配，利用cache
ConcCGCThreads开始时各自忙着自己平均分配下来的Region，如果有线程先忙完了，会尝试“偷”其他线程还没做的Region来干活(Fork/Join 双链表队列)

### Spring Boot执行
run 方法执行流程

创建计时器，用于记录SpringBoot应用上下文的创建所耗费的时间。
开启所有的SpringApplicationRunListener监听器，用于监听Sring Boot应用加载与启动信息。
创建应用配置对象(main方法的参数配置) ConfigurableEnvironment

创建要打印的Spring Boot启动标记 Banner

创建 ApplicationContext应用上下文对象，web环境和普通环境使用不同的应用上下文。
创建应用上下文启动失败原因分析对象 FailureAnalyzers

刷新应用上下文，并从xml、properties、yml配置文件或数据库中加载配置信息，并创建已配置的相关的单例bean。到这一步，所有的非延迟加载的Spring bean都应该被创建成功。
调用实现了*Runner类型的bean的run方法，开始应用启动。
完成Spring Boot启动监听
打印Spring Boot上下文启动耗时
如果在上述步骤中有异常发生则日志记录下才创建上下文失败的原因并抛出IllegalStateException异常。

https://www.jianshu.com/p/d24f7891b5db

### Final
finally是在return后面的表达式运算后执行的（此时并没有返回运算后的值，而是先把要返回的值保存起来，管finally中的代码怎么样，返回的值都不会改变，任然是之前保存的值），所以函数返回值是在finally执行前确定的；
任何执行try 或者catch中的return语句之前，都会先执行finally语句，如果finally存在的话。
如果finally中有return语句，那么程序就return了，所以finally中的return是一定会被return的，
编译器把finally中的return实现为一个warning。


### 类加载顺序
启动类
扩展类
系统类
自定义类

静态代码段
静态变量
静态方法