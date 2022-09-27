# Kafka


<!--more-->



# Kafka

​		Kafka是一个开源的分布式事件流平台(Event Streaming Platform)，广泛应用于高性能数据管道、流分析、数据集成和关键任务应用;



## 消息队列

​		企业中比较常见的消息队列产品主要有Kafka、ActiveMQ、RabbitMQ、RocketMQ等。在大数据场景主要采用Kafka作为消息队列

#### 消息队列的模式

1. 点对点模式:  消费者主动拉取数据、消息收到后清除消息

   ![截屏2022-09-03 下午9.02.33](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%889.02.33.png)

2. 发布/订阅模式

   - 可以有多个topic主题(浏览、点赞、收藏、评论等) 
   - 消费者消费数据之后，不删除数据
   - 每个消费者相互独立，都可以消费到数据

   ![截屏2022-09-03 下午9.05.13](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%889.05.13.png)

3. 

## 应用场景

传统的消息队列的主要应用场景包括：缓存/消峰**、**解耦**和**异步通信。

### 缓存/消峰

​		有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况![截屏2022-09-03 下午8.58.05](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%888.58.05.png)

### 解耦

​		允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。![截屏2022-09-03 下午8.59.54](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%888.59.54.png)

### 异步通信

​		允许用户把一个消息放入队列，但并不立即处理它，然后在需要的时候再去处理它们。![截屏2022-09-03 下午9.00.38](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%889.00.38.png)



## Kafka架构

![截屏2022-09-03 下午9.09.41](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%889.09.41.png)

1. Producer：消息生产者，就是向Kafka broker发消息的客户端;
2. Consumer：消息消费者，向Kafka broker取消息的客户端。
3. **Consumer Group（CG）：消费者组，由多个consumer组成。**消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。**所有的消费者都属于某个消费者组，即**消费者组是逻辑上的一个订阅者。
4. Broker：一台Kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
5. Topic：主题, 可以理解为一个队列，生产者和消费者面向的都是一个topic。
6. **Partition**：**为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，**一个topic可以分为多个partition，每个partition是一个有序的队列。
7. **Replica**：**副本。一个topic的每个分区都有若干个副本，一个**Leader**和若干个**Follower。
8. **Leader**：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是Leader;
9. **Follower**：每个分区多个副本中的“从”，实时从Leader中同步数据，保持和Leader数据的同步。Leader发生故障时，某个Follower会成为新的Leader。



## 生产者

### 生产者消息发送原理

​		在消息发送的过程中，涉及到了两个线程——**main** 线程和 **Sender** 线程。在 main 线程 中创建了一个双端队列 **RecordAccumulator**。main 线程将消息发送给 RecordAccumulator， Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker。![截屏2022-09-03 下午9.26.11](https://raw.githubusercontent.com/NoobMidC/pics/main/截屏2022-09-03 下午9.26.11.png)

​			主线程中，由KafkaProducer创建消息，然后通过可能的拦截器、序列化器和分区器的作用，缓存消息到消息加载器（RecordAccumulator，也称为消息收集器）中，Sender线程负责从消息加载器（RecordAccumulator）中获取消息并将其发送到Kafka中。

> 拦截器: 对数据进行加工和操作; 如流量拦截和放行处理,如实现白名单
>
> 序列化器: 将生产出的消息kv序列化为标准协议
>
> 分区器: 根据其存储的元数据, 确定消息发送到哪个分区

- 消息加载器RecordAccumulator (内存中)

  ​		消息加载器（RecordAccumulator）主要用来缓存消息以便Sender线程可以批量发送，进而减少网络传输的资源消耗以提升性能。缓存大小默认为32M; 每个DQuene为发给某个partition的数据队列(一个分区对应一个队列); 即ProducerBatch;	当满足以下两个条件的任意一个之后，消息由sender线程发送。

  1. 一个队列中的消息累计达到batch.size，默认是16kb。
  2. 等待时间达到linger.ms，默认是0毫秒。

  > RecordAccumulator 中维护一个内存池;  dqueue有数据时请求内存池, 数据发送完毕则将内存归还内存池; 避免了内存的频繁申请和释放;

- sender线程

  ​		Sender线程首先会通过sender读取数据，并创建发送的请求，针对Kafka集群里的每一个Broker，都会有一个InFlightRequests请求队列存放在NetWorkClient中，默认每个InFlightRequests请求队列中缓存5个请求。接着这些请求就会通过Selector发送到Kafka集群中。

  

- kafka集群

  ​		当请求发送到发送到Kafka集群后，Kafka集群会返回对应的acks信息。生产者可以根据具体的情况选择处理acks信息。比如是否需要等有回应之后再继续发送消息，还是不管发送成功失败都继续发送消息。

  1. 如果成功;  InFlightRequests清理掉对应请求; DQuene清理掉对应数据
  2. 如果失败; 则进行重试retires次数, 默认为int最大值 即2147483647



### 生产者消息发送方式

- 异步发送
- 带回调的异步发送
- 同步发送



### 生产者分区

#### 分区的优势

1. **便于合理使用存储资源**;  每个Partition在一个Broker上存储，可以把海量的数据按照分区切割成一

   块一块数据存储在多台Broker上。合理控制分区的任务，可以实现负载均衡的效果。

2. 提高并行度，生产者可以以分区为单位发送数据;消费者可以以分区为单位进行消费数据。

![截屏2022-09-05 下午12.06.27](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%8812.06.27.png)



#### 分区策略

1. 指明**partition**的情况下，直接将指明的值作为**partition**值; 例如partition=0，所有数据写入 分区0
2. 没有指明**partition**值但有**key**的情况下，将**key**的**hash**值与**topic**的 **partition**数进行取余得到**partition**值;
    例如:**key1**的**hash**值**=**5**， **key2**的**hash**值**=6，**topic**的**partition**数**=2**，那 么**key1** 对应的**value1**写入**1**号分区，**key2**对应的**value2**写入**0**号分区。
3. 既没有**partition**值又没有**key**值的情况下，**Kafka**采用**Sticky Partition**(黏性分区器)，会随机选择一个分区，并尽可能一直使用该分区，待该分区的**batch**已满或者到linger.ms时间到之后，**Kafka**再随机一个分区进行使用(和上一次的分区不同)。例如:第一次随机选择**0**号分区，等**0**号分区当前批次满了(默认**16k**)或者**linger.ms**设置的时间到， **Kafka**再随机一个分区进行使用(如果还是**0**会继续随机)。



### 生产者优化

#### 提高吞吐量

1. batch.size: 批次大小, 默认为16k, 可以扩大; 
2. linger.ms:等待时间，修改为5-100ms; 
3. compression.type:设置为snappy, 数据压缩后传输的效率提高了
4. RecordAccumulator:缓冲区大小，修改为64m



#### 数据可靠性

ACK应答级别

![截屏2022-09-05 下午12.17.23](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%8812.17.23.png)

![截屏2022-09-05 下午12.20.00](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%8812.20.00.png)

可靠性总结: 

- **acks=0**，生产者发送过来数据就不管了，可靠性差，效率高;

- **acks=1**，生产者发送过来数据**Leader**应答，可靠性中等，效率中等;

- **acks=-1**，生产者发送过来数据**Leader**和**ISR**队列里面所有**Follwer**应答，可靠性高，效率低;

  ​		在生产环境中，**acks=0**很少使用;**acks=1**，一般用于传输普通日志，允许丢个别数据;**acks=-1**，一般用于传输和钱相关的数据， 对可靠性要求比较高的场景。



#### 数据去重

数据重复发送的原因

​		在ack = -1 时,需要isr中所有的所有节点应答后才返回客户端成功; 如果在某个follower接收到一条数据后, leader挂掉;然后这个follower被选为leader; 由于集群未应答sender成功, 因此selector回重试,这样现在的leader就会有两条相同的数据;

![截屏2022-09-05 下午12.34.23](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%8812.34.23.png)

- 数据传输语义
  1. 至少一次(保证数据不丢失,不能保证重复性): 最少发送一次,如果失败则继续发送;   **ACK级别设置为-1+分区副本大于等于2+ISR里应答的最小副本数量大于等于2**
  2. 至多一次(不保证数据不丢失,能保证数据不重复): 最多发送一次, 失败成功无所谓; **ACK级别设置为0**
  3. 精确一次: 对于一些非常重要的信息，比如和钱相关的数据，要求数据既不能重复也不丢失; **可以使用幂等性**



- 幂等性

  ​		幂等性就是指Producer不论向Broker发送多少次重复数据，Broker端都只会持久化一条，保证了不重复。

  ​		精确一次(**Exactly Once**) **=** 幂等性 **+** 至少一次( **ack=-1 +** 分区副本数**>=2 + ISR**最小副本数量**>=2**) ;

  ​		重复数据的判断标准:具有<PID, Partition, SeqNumber>相同主键的消息提交时，Broker只会持久化一条。其中PID是Kafka每次重启都会分配一个新的;Partition 表示分区号;Sequence Number是单调自增的。

  ​		所以幂等性只能保证的是**在单分区单会话内不重复**。

![截屏2022-09-05 下午12.43.18](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%8812.43.18.png)

​			幂等性使用: 开启参数 **enable.idempotence** 默认为 true，false 关闭。



#### 生产者事务

​		开启事务，必须开启幂等性。

![截屏2022-09-05 下午12.44.57](https://raw.githubusercontent.com/NoobMidC/pics/main/截屏2022-09-05 下午12.44.57.png)

#### 事务

- 使用场景
  1. **生产者发送多条消息可以封装在一个事务中，形成一个原子操作。**多条消息要么都发送成功，要么都发送失败。
  2. **read-process-write模式：将消息消费和生产封装在一个事务中，形成一个原子操作。**在一个流式处理的应用中，常常一个服务需要从上游接收消息，然后经过处理后送达到下游，这就对应着消息的消费和生成。
- 事务配置
  1. 对于Producer，需要设置`transactional.id`属性，这个属性的作用下文会提到。设置了`transactional.id`属性后，`enable.idempotence`属性会自动设置为true。
  2. 对于Consumer，需要设置`isolation.level = read_committed`，这样Consumer只会读取已经提交了事务的消息。另外，需要设置`enable.auto.commit = false`来关闭自动提交Offset功能。
- 事务特性

1. 原子写

   ​          Kafka的事务特性本质上是支持了Kafka跨分区和Topic的原子写操作。在同一个事务中的消息要么同时写入成功，要么同时写入失败。我们知道，Kafka中的Offset信息存储在一个名为_consumed_offsets的Topic中，因此read-process-write模式，除了向目标Topic写入消息，还会向_consumed_offsets中写入已经消费的Offsets数据。因此read-process-write本质上就是跨分区和Topic的原子写操作。**Kafka的事务特性就是要确保跨分区的多个写操作的原子性。**

   

2. 拒绝僵尸实例（Zombie fencing)

   ​		在分布式系统中，一个instance的宕机或失联，集群往往会自动启动一个新的实例来代替它的工作。此时若原实例恢复了，那么集群中就产生了两个具有相同职责的实例，此时前一个instance就被称为“僵尸实例（Zombie Instance）”。在Kafka中，两个相同的producer同时处理消息并生产出重复的消息（read-process-write模式），这样就严重违反了精确一次(Exactly Once Processing)的语义。这就是僵尸实例问题。

   ​		Kafka事务特性通过`transaction-id`属性来解决僵尸实例问题。所有具有相同`transaction-id`的Producer都会被分配相同的pid，同时每一个Producer还会被分配一个递增的epoch。Kafka收到事务提交请求时，如果检查当前事务提交者的epoch不是最新的，那么就会拒绝该Producer的请求。从而达成拒绝僵尸实例的目标。

   

3. 读事务消息

   ​               为了保证事务特性，Consumer如果设置了`isolation.level = read_committed`，那么它只会读取已经提交了的消息。在Producer成功提交事务后，Kafka会将所有该事务中的消息的`Transaction Marker`从`uncommitted`标记为`committed`状态，从而所有的Consumer都能够消费。



- 事务原理

1. **查找Tranaction Corordinator**

   ​         Producer向任意一个brokers发送 FindCoordinatorRequest请求来获取Transaction Coordinator的地址。

   

2. **初始化事务 initTransaction**

   ​      Producer发送InitpidRequest给Transaction Coordinator，获取pid。**Transaction Coordinator在Transaciton Log中记录这<TransactionId,pid>的映射关系。**另外，它还会做两件事：

   - 恢复（Commit或Abort）之前的Producer未完成的事务
   - 对PID对应的epoch进行递增，这样可以保证同一个app的不同实例对应的PID是一样，而epoch是不同的。

   

3. **开始事务beginTransaction**

   ​     执行Producer的beginTransacion()，它的作用是Producer在**本地记录下这个transaction的状态为开始状态。**这个操作并没有通知Transaction Coordinator，因为Transaction Coordinator只有在Producer发送第一条消息后才认为事务已经开启。

   

4. **read-process-write流程**

   ​		一旦Producer开始发送消息，**Transaction Coordinator会将该<Transaction, Topic, Partition>存于Transaction Log内，并将其状态置为BEGIN**。另外，如果该<Topic, Partition>为该事务中第一个<Topic, Partition>，Transaction Coordinator还会启动对该事务的计时（每个事务都有自己的超时时间）。

   ​		在注册<Transaction, Topic, Partition>到Transaction Log后，生产者发送数据，虽然没有还没有执行commit或者abort，但是此时消息已经保存到Broker上了。即使后面执行abort，消息也不会删除，只是更改状态字段标识消息为abort状态。

   

5. **事务提交或终结 commitTransaction/abortTransaction**

   ​		在Producer执行commitTransaction/abortTransaction时，Transaction Coordinator会执行一个两阶段提交：

   - **第一阶段，将Transaction Log内的该事务状态设置为`PREPARE_COMMIT`或`PREPARE_ABORT`**
   - **第二阶段，将`Transaction Marker`写入该事务涉及到的所有消息（即将消息标记为`committed`或`aborted`）。**这一步骤Transaction Coordinator会发送给当前事务涉及到的每个<Topic, Partition>的Leader，Broker收到该请求后，会将对应的`Transaction Marker`控制信息写入日志。

   ​      一旦`Transaction Marker`写入完成，Transaction Coordinator会将最终的`COMPLETE_COMMIT`或`COMPLETE_ABORT`状态写入Transaction Log中以标明该事务结束。

   

   > 总结: 
   >
   > - Transaction Marker与PID提供了识别消息是否应该被读取的能力，从而实现了事务的隔离性。
   > - Offset的更新标记了消息是否被读取，从而将对读操作的事务处理转换成了对写（Offset）操作的事务处理。
   > - Kafka事务的本质是，将一组写操作（如果有）对应的消息与一组读操作（如果有）对应的Offset的更新进行同样的标记（Transaction Marker）来实现事务中涉及的所有读写操作同时对外可见或同时对外不可见。
   > - Kafka只提供对Kafka本身的读写操作的事务性，不提供包含外部系统的事务性。



#### 数据有序

![截屏2022-09-05 下午1.42.29](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%881.42.29.png)

1. 在1.x以前保证数据单分区有序; 需要设置每个连接中的InFlightRequests队列中的请求数(**max.in.flight.requests.per.connection**)为1

2. 1.x以后, 不开启幂等性时和上面一样; 开启后设置其小于等于5;

   > 因为在kafka1.x以后，启用幂等后，kafka服务端会缓存producer发来的最近5个request的元数据， 故无论如何，都可以保证最近5个request的数据都是有序的。





## Broker

### Zookeeper 存储的信息

![截屏2022-09-05 下午1.52.14](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%881.52.14.png)

 

### Broker工作流程

![截屏2022-09-05 下午1.55.48](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%881.55.48.png)

1. 注册: 每次启动一个broker都会到zk中注册节点信息;

2. 选择controller节点:  所以节点的controller去zk抢注册; 谁先注册谁为controller_leader;

3. 监听: 选举出来的controller 监听所有broker的状态

4. 选举: controller_leader决定leader选举;

   > 选举规则: 在ISR中存活为前提, 按照AR排在前面的优先; 如AR[1,0,2], ISR为[1,0,2],则leader会按照1,0,2顺序轮询

5. 同步信息: controller上传节点信息到zk,其他controller从zk同步相关信息

6. 重新选举: 如果leader挂掉, controller监听到节点变化, 向zk拉取节点信息(ISR等),并重新选举leader

7. 更新leader和ISR

> replica.lag.time.max.ms: ISR中，如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值，默认30s。



### 开发中的节点服役退役操作

#### 节点服役

1. 启动新节点
2. 执行负载均衡计划,将数据重分配到每个broker上
3. 执行副本存储计划, 重新分配主分区的副本位置

节点退役

1. 执行负载均衡计划, 将数据重分配到后续存活的broker上
2. 执行副本存储计划, 重新分配主分区的副本位置
3.  关闭旧节点



### Kafka副本

#### 基本信息

1. Kafka副本作用：提高数据可靠性。
2. Kafka默认副本1个，生产环境一般配置为2个，保证数据可靠性；太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。
3. Kafka中副本分为：Leader和Follower。Kafka生产者只会把数据发往Leader，然后Follower找Leader进行同步数据。
4. Kafka分区中的所有副本统称为AR（Assigned Repllicas）。

>  AR = ISR + OSR
>
> **ISR**，表示和Leader保持同步的Follower集合。如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由**replica.lag.time.max.ms**参数设定，默认30s。Leader发生故障之后，就会从ISR中选举新的Leader。
>
> **OSR**，表示Follower与Leader副本同步时，延迟过多的副本。



#### Leader选举流程

​		Kafka集群中有一个broker的Controller会被选举为Controller Leader，负责管理集群broker的上下线，所有topic的分区副本分配和Leader选举等工作; Controller的信息同步工作是依赖于Zookeeper的

![截屏2022-09-05 下午2.18.38](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.18.38.png)



#### 故障处理

- Follower故障处理

![截屏2022-09-05 下午2.19.25](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.19.25.png)

- Leader故障处理

![截屏2022-09-05 下午2.20.14](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.20.14.png)



#### 分区副本分配

![截屏2022-09-05 下午2.25.04](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.25.04.png)

1. leader尽可能错开, 第一个分配了则第二个broker, 循环往复
2. follower第一批和leader不间隔, 第二批和leader间隔一个broker, 后面递增
3. 可以手动创建topic并指定每个分区的副本位置



#### Leader Partition负载平衡

​		正常情况下，Kafka本身会自动把Leader Partition均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某 些broker宕机，会导致Leader Partition过于集中在其他少部分几台broker上，这会导致少数几台broker的读写请求压力过高，其他宕机的 broker重启之后都是follower partition，读写请求很低，造成集群负载不均衡。

> - auto.leader.rebalance.enable，默认是true。 自动Leader Partition 平衡
> - leader.imbalance.per.broker.percentage， 默认是10%。每个broker允许的不平衡 的leader的比率。如果每个broker超过 了这个值，控制器会触发leader的平衡。
> - leader.imbalance.check.interval.seconds， 默认值300秒。检查leader负载是否平衡 的间隔时间。

![截屏2022-09-05 下午2.32.48](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.32.48.png)



#### 增加副本因子

​		在生产环境当中，由于某个主题的重要等级需要提升，我们考虑增加副本。副本数的增加需要先制定计划，然后根据计划执行。

1. 创建topic

2. 手动增加副本存储

   （1）创建副本存储计划（所有副本都指定存储在broker0、broker1、broker2中）。

   （2）执行副本存储计划。





### 文件存储(持久化)

##### 文件存储机制

​		Topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是Producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制， 将每个partition分为多个segment。每个segment包括:“.index”文件、“.log”文件和.timeindex等文件。这些文件位于一个文件夹下，该 文件夹的命名规则为:topic名称+分区序号，例如:first-0。![截屏2022-09-05 下午2.38.29](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.38.29.png)

- **index**文件和log文件详解

![截屏2022-09-05 下午2.41.47](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.41.47.png)



### 文件清除策略

Kafka中默认的日志保存时间为7天，可以通过调整如下参数修改保存时间。

- log.retention.hours，最低优先级小时，默认7天。
- log.retention.minutes，分钟。
- log.retention.ms，最高优先级毫秒。
- log.retention.check.interval.ms，负责设置检查周期，默认5分钟

#### 清除方式

Kafka中提供的日志清理策略有delete和compact两种。

- delete: 将过期数据删除

log.cleanup.policy = delete  所有数据启用删除策略

1. 基于时间：默认打开。以segment中所有记录中的最大时间戳作为该文件时间戳。这样就能防止segment中部分超时而删除所有数据的情况;

2. 基于大小：默认关闭。超过设置的所有日志总大小，删除最早的segment。log.retention.bytes，默认等于-1，表示无穷大。

- compact日志压缩

![截屏2022-09-05 下午2.46.17](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%882.46.17.png)





### 高效读写数据

1. **Kafka**本身是分布式集群，可以采用分区技术，并行度高

2. **读数据采用稀疏索引，可以快速定位要消费的数据**

3. **顺序写磁盘**

   ​		Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写

4. **页缓存** **+** **零拷贝技术**

   - 零拷贝:Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就不用走应用层，传输效率高。

   - **PageCache**页缓存: Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存 都当做了磁盘缓存来使用。

> log.flush.interval.messages: 强制页缓存刷写到磁盘的条数，默认是long的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。
>
> log.flush.interval.ms: 每隔多久，刷数据到磁盘，默认是null。一般不建议修改，交给系统自己管理。





## 消费者

### 消费模式

1. **pull**(拉)模式

   consumer采用从broker中主动拉取数据。**Kafka**采用这种方式

2. **push**(推)模式

   Kafka没有采用这种方式，因为由broker 决定消息发送速率，很难适应所有消费者的消费速率;

> pull模式不足之处是，如果Kafka没有数 据，消费者可能会陷入循环中，一直返回 空数据。



### 工作流程

![截屏2022-09-05 下午3.00.32](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.00.32.png)

1. 不同的消费者之前互不影响
2. 一个分区只能由消费者组中的一个消费者消费; 一个消费者可以消费多个分区
3. 消费者的offset由消费者提交给_consumer_offsets主题保存

> 1.x以前将offset存储在zk中, 由于需要储存每个消费者offset,每次请求都需要向zk拿取offset, 其整个网络传输的负担太大



### 消费者组原理

Consumer Group(CG):消费者组，由多个consumer组成。形成一个消费者组的条件，是所有消费者的groupid相同。 

- 消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费。
-  消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
- 如果向消费组中添加更多的消费者，超过 主题分区数量，则有一部分消费者就会闲 置，不会接收任何消息。
- 消费者组之间互不影响。所 有的消费者都属于某个消费者组，即消费者组是逻辑上 的一个订阅者。



#### 消费者组初始化流程

1. coordinator: 辅助实现消费者组的初始化和分区的分配。
    coordinator节点选择 = groupid的hashcode值 % 50(consumer_offsets的分区数量)
    	例如: groupid的hashcode值 = 1，1% 50 = 1，那么consumer_offsets 主题的1号分区，在哪个broker上，就选择这个节点的coordinator;作为这个消费者组的老大。消费者组下的所有的消费者提交offset的时候就往这个分区去提交offset。
2. 选出一个consumer作为leader
3. coordinator把要消费的topic信息发给leader
4. leader制定消费方案(即哪些消费者消费哪些分区);并将方案发给coordinator;
5. coordinator下发消费方案给每个消费者
6. 消费者和coordinator保持心跳(默认3s),一旦超时(45s),消费者会被移除,触发再平衡; 或者消费者处理消息时间过长(5min),也会触发再平衡

![截屏2022-09-05 下午3.09.32](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.09.32.png)



#### 详细消费过程

![截屏2022-09-05 下午3.17.21](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.17.21.png)

1. 发送抓取数据的请求

   > - 每批次最小的抓取大小,默认为一字节
   > - 一个批次数据最小未到达超时时间为500ms(未达到数据大小也拉取)
   > - 每批次最大抓取大小默认为50m

2. 成功回调将数据发给消费者,放在消息队列中;

3.  消费者每次拉取500条数据进行处理(反序列化、拦截器、处理数据)



### 分区分配策略

​		Kafka有四种主流的分区分配策略: Range、RoundRobin、Sticky、CooperativeSticky。 可以通过配置参数partition.assignment.strategy，修改分区的分配策略。默认策略是**Range+ CooperativeSticky**。Kafka可以同时使用多个分区分配策略

#### Range策略

- 原理

Range 是对每个 topic 而言的。

1. 首先对同一个 topic 里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序

   > 假如现在有 7 个分区，3 个消费者，排序后的分区将会是0,1,2,3,4,5,6;消费者排序完之后将会是C0,C1,C2

2. 通过 partitions数/consumer数 来决定每个消费者应该 消费几个分区。如果除不尽，那么前面几个消费者将会多 消费 1 个分区。

   > 例如，7/3 = 2 余 1 ，除不尽，那么 消费者 C0 便会多 消费 1 个分区。 8/3=2余2，除不尽，那么C0和C1分别多 消费一个。

**容易产生数据倾斜!**![截屏2022-09-05 下午3.28.34](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.28.34.png)

- 再分配

  ​		一个消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。

  1. 消费者挂掉后其任务会整体被分配到 某一个消费者
  2. 再次重新发送消息, 挂掉的被踢出消费者组，然后重新按照 range 方式分配。

#### **RoundRobin**策略

- 原理

​			RoundRobin 轮询分区策略，是把所有的 partition 和所有的 consumer 都列出来，然后按照 hashcode 进行排序，最后 通过轮询算法来分配 partition 给到各个消费者。![截屏2022-09-05 下午3.32.07](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.32.07.png)

- 再分区

1. 消费者挂掉后其任务会按照 RoundRobin 的方式，把数据轮询分成多个分区数据，分别其他的消费者消费。
2. 再次重新发送消息, 挂掉的被踢出消费者组，然后重新按照 range 方式分配。



#### **Sticky** 策略

​		粘性分区定义:可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前， 考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。

​		粘性分区是 Kafka 从 0.11.x 版本开始引入这种分配策略，首先会尽量均衡的放置分区 到消费者上面，在出现同一消费者组内消费者出现问题的时候，会尽量保持原有分配的分 区不变化。

- 再平衡案例	

  1. 按照粘性规则，尽可能均衡的随机分成 多个分区数据，分别

     由 剩余的消费者消费

  2. 再次重新发送消息, 挂掉的被踢出消费者组，然保持a步骤中的分区分配不变

  





### Offset

#### 查看消费者offset

​		在配置文件 config/consumer.properties 中添加配置 exclude.internal.topics=false， 默认是 true，表示不能消费系统主题。为了查看该系统主题数据，所以该参数修改为 false

然后可以指定topic为__consumer_offsets进行查看

```sh
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh --topic __consumer_offsets --bootstrap-server hadoop102:9092 -- consumer.config config/consumer.properties --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageForm atter" --from-beginning
```

#### 自动提交

-  **enable.auto.commit**:是否开启自动提交offset功能，默认是true
-  **auto.commit.interval.ms**:自动提交offset的时间间隔，默认是5s

![截屏2022-09-05 下午3.44.09](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.44.09.png)





#### 手动提交

​		手动提交offset的方法有两种:分别是commitSync(同步提交)和commitAsync(异步提交)。两者的相 同点是，都会将本次提交的一批数据最高的偏移量提交;不同点是，同步提交阻塞当前线程，一直到提交成 功，并且会自动失败重试(由不可控因素导致，也会出现提交失败);而异步提交则没有失败重试机制，故有可能提交失败。

-  commitSync(同步提交):必须等待offset提交完毕，再去消费下一批数据。
-  commitAsync(异步提交) :发送完提交offset请求后，就开始消费下一批数据了。

![截屏2022-09-05 下午3.46.47](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.46.47.png)



#### **漏消费和重复消费**

- **重复消费：**已经消费了数据，但是offset没提交。
- **漏消费：**先提交offset后消费，有可能会造成数据的漏消费。

1. 重复消费: 自动提交引起

![截屏2022-09-05 下午3.49.30](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.49.30.png)

2. 漏消费: 漏消费。设置offset为手动提交，当offset被提交时，数据还在内存中未落盘，此时刚好消费者线程被kill掉，那么offset已经提交，但是数据未处理，导致这部分内存中的数据丢失

   ![截屏2022-09-05 下午3.51.15](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.51.15.png)



### 消费者事务

​			如果想完成Consumer端的精准一次性消费，那么需要Kafka消费端将消费过程和提交offset 过程做原子绑定。此时我们需要将Kafka的offset保存到支持事务的自定义介质(比如 MySQL)

![截屏2022-09-05 下午3.52.12](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.52.12.png)





### 消费者如何提高吞吐量

1. 如果是Kafka消费能力不足，则可以考虑增加Topic的分区数，并且同时提升消费组的消费者 数量，消费者数 = 分区数。(两者缺一不可)
2. 如果是下游的数据处理不及时:提高每批次拉取的数量。批次拉取数据过少(拉取数据/处理时间 < 生产速度)， 使处理的数据小于生产的数据，也会造成数据积压。

![截屏2022-09-05 下午3.54.02](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.54.02.png)









## Kraft模式

![截屏2022-09-05 下午3.55.24](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-05%20%E4%B8%8B%E5%8D%883.55.24.png)

​		左图为 Kafka 现有架构，元数据在 zookeeper 中，运行时动态选举 controller，由 controller 进行 Kafka 集群管理。右图为 kraft 模式架构(实验性)，不再依赖 zookeeper 集群， 而是用三台 controller 节点代替 zookeeper，元数据保存在 controller 中，由 controller 直接进 行 Kafka 集群管理。

优点: 

-  Kafka不再依赖外部框架，而是能够独立运行;
-  controller管理集群时，不再需要从zookeeper中先读取数据，集群性能上升;
-  由于不依赖zookeeper，集群扩展时不再受到zookeeper读写能力限制;
-  controller 不再动态选举，而是由配置文件规定。这样我们可以有针对性的加强
- controller 节点的配置，而不是像以前一样对随机 controller 节点的高负载束手无策。







## 面试题

### 1. **Kafka** 是什么?主要应用场景有哪些?

Kafka 是一个分布式流式处理平台。 流平台具有三个关键功能:

1. 消息队列:发布和订阅消息流，这个功能类似于消息队列，这也是 Kafka 也被归类为消息队列的原因。 
2. 容错的持久方式存储记录消息流:Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。 
3. 流式处理平台:在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。

Kafka 主要有两大应用场景: 

1. 消息队列:建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. 数据处理:构建实时的流数据处理程序来转换或处理数据流。



### 2. 和其他消息队列相比，**Kafka** 的优势在哪里?

- 极致的性能:基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千 万级别的消息。
-  生态系统兼容性无可匹敌:Kafka 与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。



### 3. **Kafka** 的多副本机制了解吗?

​		Kafka 为分区(Partition)引入了多副本(Replica)机制。分区(Partition)中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发 送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。

​		生产者和消费者只与 leader 副本交互。你可以理解为其他副本只是 leader 副本的拷贝，它们的存在只是为了保证 消息存储的安全性。当 leader 副本发生故障时会从 follower 中选举出一个 leader,但是 follower 中如果有和 leader 同步程度达不到要求的参加不了 leader 的竞选。

​		优势: 

1. Kafka 通过给特定 Topic 指定多个 Partition,而各个 Partition 可以分布在不同的 Broker 上,这样便能提供比较 好的并发能力(负载均衡)。
2. Partition 可以指定对应的 Replica 数,这也极大地提高了消息存储的安全性,提高了容灾能力，不过也相应的增 加了所需要的存储空间。



### 4. **Zookeeper** 在 **Kafka** 中的作用知道吗?

- Broker 注册:在 Zookeeper 上会有一个专门用来进行 Broker 服务器列表记录的节点。每个 Broker 在启动 时，都会到 Zookeeper 上进行注册，即到/brokers/ids 下创建属于自己的节点。每个 Broker 就会将自己的 IP 地址和端口等信息记录到该节点中去
- Topic 注册:在 Kafka 中，同一个 Topic 的消息会被分成多个分区并将其分布在多个 Broker 上，这些分区信 息及与 Broker 的对应关系也都是由 Zookeeper 在维护。比如我创建了一个名字为 my-topic 的主题并且它有 两个分区，对应到 zookeeper 中会创建这些文件夹:/brokers/topics/my- topic/Partitions/0、/brokers/topics/my - topic/Partitions/1
- 负载均衡:上面也说过了 Kafka 通过给特定 Topic 指定多个 Partition,而各个 Partition 可以分布在不同的 Broker 上,这样便能提供比较好的并发能力。对于同一个 Topic 的不同 Partition，Kafka 会尽力将这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里 面。当 Consumer 消费的时候，Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态 负载均衡。
- 总而言之, 协助集群管理; 保存了集群的broker元数据、topic元数据, 需要时controller从这里拿取;



### 5.**Kafka** 如何保证消息的消费顺序?

​			每次添加消息到 Partition(分区)的时候都会采用尾加法; Kafka 只能为我们保证 Partition(分区)中的 消息有序。消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量;

​		Kafka 通过偏移量(offset)来保证消息在分区内的顺序性。所以，我们就有一种很简单的保证消 息消费顺序的方法:1 个 Topic 只对应一个 Partition。这样当然可以解决问题，但是破坏了 Kafka 的设计初 衷。 Kafka 中发送1 条消息的时候，可以指定 topic, partition, key,data(数据)4 个参数。如果你发送消息 的时候指定了 Partition 的话，所有消息都会被发送到指定的 Partition。并且，同一个 key 的消息可以保证只 发送到同一个 partition，这个我们可以采用表/对象的 id 来作为 key。

总结: 

1. 1 个 Topic 只有一个 Partition。
2. 发送消息的时候指定 key/Partition。



### 6. **Kafka** 如何保证消息不丢失?

1. 副本机制
2. 生产者应答机制,设置为-1, 则需要所有ISR同步后返回确认信息; 失败则重试;



### 7. 判断节点活着的条件

1. 节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接;
2. 如果Follower长时间未向Leader发或同步数据，则该Follower将被踢出ISR。该时间阈值，默认30s。



### 8. **Kafka consumer** 是否可以消费指定分区消息吗?

​		Kafa consumer 消费消息时，**向 broker 发出"fetch"请求去消费特定分区的消息**，consumer 指定消息在日志中的 偏移量(offset)，就可以消费从这个位置开始的消息，customer 拥有了 offset 的控制权，可以向后回滚去重新 消费之前的消息



### 9. **Kafka** 高效文件存储设计特点是什么?

- Kafka 把 topic 中一个 parition 大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
-  通过索引信息可以快速定位 message 和确定 response 的最大大小。
-  通过 index 元数据全部映射到 memory，可以避免 segment file 的 IO 磁盘操作。 
- 通过索引文件稀疏存储，可以大幅降低 index 文件元数据占用空间大小。



### 10. **partition** 的数据如何保存到硬盘?

- topic 中的多个 partition 以文件夹的形式保存到 broker，每个分区序号从0 递增，且消息有序。 Partition 文件下有多个 segment(xxx.index，xxx.log)

- segment 文件里的大小和配置文件大小一致可以根据要求修改，默认为1g。如果大小大于1g 时，会滚动一个新的 segment 并且以上一个 segment 最后一条消息的偏移量命名。



### 11. **kafka** 维护消费状态跟踪的方法有什么?

offset存储在broker的consumer_offset主题中

1. 手动提交offset; 有重复消费的问题
2. 自动提交offset; 有漏消费的问题



### 12. **Kafka** 为什么那么快**?**

1. **Kafka**本身是分布式集群，可以采用分区技术，并行度高

2. **读数据采用稀疏索引，可以快速定位要消费的数据**

3. **顺序写磁盘**

   ​		Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写

4. **页缓存** **+** **零拷贝技术**

   - 零拷贝: Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就不用走应用层，传输效率高。

   - **PageCache**页缓存: Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存 都当做了磁盘缓存来使用。



### 13. **Kafka** 的流处理是什么意思?

​		连续、实时、并发和以逐记录方式处理数据的类型，我们称之为 Kafka 流处理。



### 14. 延迟队列实现

#### Redis ZSet

​		我们知道 Redis 有一个有序集合的数据结构 ZSet，ZSet 中每个元素都有一个对应 Score，ZSet 中所有元素是按照其 Score 进行排序的。

  1. 入队操作：`ZADD KEY timestamp task`, 我们将需要处理的任务，按其需要延迟处理时间作为 `Score` 加入到 `ZSet` 中。Redis 的 `ZAdd` 的时间复杂度是`O(logN)`，N是 ZSet 中元素个数，因此我们能相对比较高效的进行入队操作。

  2. 起一个进程定时（比如每隔一秒）通过`ZREANGEBYSCORE`方法查询 ZSet 中 `Score` 最小的元素，具体操作为：`ZRANGEBYSCORE KEY -inf +inf limit 0 1 WITHSCORES`。查询结果有两种情况：

     1. 查询出的分数小于等于当前时间戳，说明到这个任务需要执行的时间了，则去异步处理该任务；
     2. 查询出的分数大于当前时间戳，由于刚刚的查询操作取出来的是分数最小的元素，所以说明 ZSet 中所有的任务都还没有到需要执行的时间，则休眠一秒后继续查询；

     (`ZRANGEBYSCORE`操作的时间复杂度为`O(logN + M)`，其中N为 `ZSet` 中元素个数，M为查询的元素个数，因此我们定时查询操作也是比较高效的。)



​		Redis 实现延迟队列的后端架构，其在原来 Redis 的 ZSet 实现上进行了一系列的优化，使得整个系统更稳定、更健壮，能够应对高并发场景，并且具有更好的可扩展性

![在这里插入图片描述](https://raw.githubusercontent.com/NoobMidC/pics/main/watermark%252Ctype_ZmFuZ3poZW5naGVpdGk%252Cshadow_10%252Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM0NzQ0MzY%253D%252Csize_16%252Ccolor_FFFFFF%252Ct_70.png)

1. 将延迟的消息任务通过 hash 算法路由至不同的 Redis Key 上，这样做有两大好处：
   1. 避免了当一个 KEY 在存储了较多的延时消息后，入队操作以及查询操作速度变慢的问题（两个操作的时间复杂度均为`O(logN)）`。
   2. 系统具有了更好的横向可扩展性，当数据量激增时，我们可以通过增加 Redis Key 的数量来快速的扩展整个系统，来抗住数据量的增长。
2. 每个 Redis Key 都对应建立一个处理进程，称为 Event 进程，通过上述步骤 2 中所述的 ZRANGEBYSCORE 方法轮询 Key，查询是否有待处理的延迟消息。
3. 所有的 Event 进程只负责分发消息，具体的业务逻辑通过一个额外的消息队列异步处理，这么做的好处也是显而易见的：
   1. 一方面，Event 进程只负责分发消息，那么其处理消息的速度就会非常快，就不太会出现因为业务逻辑复杂而导致消息堆积的情况。
   2. 另一方面，采用一个额外的消息队列后，消息处理的可扩展性也会更好，我们可以通过增加消费者进程数量来扩展整个系统的消息处理能力。
4. Event 进程采用 Zookeeper 选主单进程部署的方式，避免 Event 进程宕机后，Redis Key 中消息堆积的情况。一旦 Zookeeper 的 leader 主机宕机，Zookeeper 会自动选择新的 leader 主机来处理 Redis Key 中的消息。

#### Rabbitmq


