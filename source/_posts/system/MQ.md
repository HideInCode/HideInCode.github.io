# MQ

> 当碰到需要解耦、异步、削峰问题时，那么就使用MQ吧。

## 常见的MQ

**ActiveMQ**：支持万级的吞吐量，较成熟完善；官方更新迭代较少，社区的活跃度不是很高，有消息丢失的情况。

**RabbitMQ**：延时低，微妙级延时，社区活跃度高，bug 修复及时，而且提供了很友善的后台界面；用 Erlang 语言开发，只熟悉 Java 的无法阅读源码和自行修复 bug。

**RocketMQ**：阿里维护的消息中间件，可以达到十万级的吞吐量，支持分布式事务。

**Kafka**：分布式的中间件，最大优点是其吞吐量高，一般运用于大数据系统的实时运算和日志采集的场景，功能简单，可靠性高，扩展性高；缺点是可能导致重复消费。

## MQ处理消息时容易遇到的问题

- 消息丢失
  - RabbitMQ
    - 生产者：
      - 开启事务（不推荐）
      - 开启confirm模式（推荐）
    - MQ：开启持久化
    - 消费者：
      - 关闭自动ACK
  - Kafka
    - 生产者：设置acks=all，数据一定不丢
    - MQ：？？
    - 消费者：Kafka会自动offset，可以关闭offset，使用手动提交offset，保证数据不会丢。
- 消息幂等
  - 写数据时先根据主键查询数据是否存在，存在则更新。
  - 利用数据库的唯一键索引。
  - 如果是redis，那么redis的set天然支持去重幂等。
- 消息顺序性
  - RabbitMQ：拆分成多个Queue，每个Queue一个Consumer，然后Consumer内部用内存队列做排队，分发给底层不同的Worker处理。
  - Kafka：写N个内存Queue，具有相同key的数据进入到同一个Queue；对于N个线程，每个线程消费一个内存Queue，保证了顺序性。
- 消息积压
  - 先修复consumer，保证消费速度，然后停掉consumer;
  - 新建个topic，partion设置为原来10倍，临时建立原来10倍的queue。
  - 写一个临时分发数据的consumer，用这个程序去消费积压数据，把数据轮询写入临时建好的10倍queue；
  - **10倍机器部署consumer**，每一批consumer消费一个queue，相当于把queue资源和consumer资源扩大10倍，以原来10倍速度消费；
  - 等消费完积压的数据，恢复原来结构，正常消费。
- 消息失效
  - 把丢失的消息查出来，重新导入MQ。

## 架构

- 如何高可用
  - RabbitMQ：镜像集群模式，队列的消息会分布在多个机器上，写消息时就会跟多个队列进行通讯。这样任意一个宕机都不影响系统。问题在于开销太大，拓展性差。
  - Kafka：partition和replica机制，每个节点都是一个broker，一个topic可以分成多个partition，partition可以存在多个broker上，也就是说一个topic的消息是分在多个机器上的。0.8版本后提供了HA机制，即replica副本机制。每个partition数据都会同步到其他机器，形成多个副本。然后所有的replica选举一个leader，生产消费都和这个leader打交道，其他的replica就是follower。写的时候leader会把数据同步到所有的follower上去，读的时候直接读leader上的数据。



