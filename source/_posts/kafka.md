---

title: kafka
top: false
cover: false
toc: true
mathjax: true
date: 2021-09-26 22:01:37
password:
summary:
tags: Java基础
categories: Java
---

# Kafka

## 1. 概念

为什么选择kafka？

1. 多生产者和多消费者。
2. 基于磁盘的数据存储。
3. 高伸缩性。
4. 高性能。



常见场景：



### 1.1 offset偏移量

* 可以唯一标识一条消息
* 偏移量决定读取数据的位置，不会有线程安全问题，消费者通过偏移量来决定下次读取的消息。
* 消息被消费之后，并不被马上删除，这样多个业务就可以重复使用kafka的消息。
* 某一个业务可以通过修改偏移量来达到重新读取消息的目的，偏移量由用户控制。
* 消息最终还是会被删除的，默认生命周期是1周（7 * 24 = 168 小时）

### 1.2 Kafka数据检索机制

* topic在物理层以partition为分组，一个topic可以分成若干个partition。

* partition还可以细分为Segment，一个partition物理上由多个Segment组成。

  * segment的参数有两个：
    * log.segment.bytes: 单个segment可容纳最大数据量，默认为1GB
    * log.segment.ms: Kafka在commit一个未写满的segment前，所等待的时间（默认为7天）

* LogSegment 文件由两部分组成，分别为“.index"和".log”文件，分别表示为Segment索引文件和数据文件。

  * partition全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset值

  * 数值大小为64位，20位数字字符长度，没有数字用0填充

  * ```
    第一个segment
    00000000000000000000.index
    00000000000000000000.log
    第二个segment，文件命名以第一个segment的最后一条消息的offset组成
    00000000000000170410.index
    00000000000000170410.log
    第三个segment，文件命名以上一个segment的最后一条消息的offset组成
    00000000000000239430.index
    00000000000000239430.log
    ```

    消息都有固定的物理结构，包括：offset(8 Bytes)、消息体的大小（4 Bytes）、magic（1Byte）等等，确定一条消息的大小，即读取到哪里截止。

    ![image-20211008154829701](C:\Users\Shawn\AppData\Roaming\Typora\typora-user-images\image-20211008154829701.png)



## 2. MQ流派

### 2.1 有Broker的MQ

这个流派通常有一台服务器作为Broker，所有的消息都通过它中转。生产者把消息发送给它就结束自己的任务了，Broker则把消息主动推送给消费者（或者消费者主动轮询）。

#### 2.1.1 重topic

kafka、JMS（ActiveMQ)就属于这个流派，生产者会发送key和数据到Broker，由Broker比较key之后决定给哪个消费者。这种模式是我们常见的模式。这种模式下topic是一种比较大的概念，甚至一个系统中就可能只有一个topic，topic某种意义上说就是queue，生产者发送key相当于说：“hi，把数据放到key的队列中”。

![image-20211009144759734](D:\1\blog\source\_posts\kafka\image-20211009144759734.png)

### 2.1.2 轻topic

RabbitMQ（或者AMQP）。生产者发送key和数据，消费者定义订阅的队列，Broker收到数据后会通过一定的逻辑计算出key对应的队列，然后把数据交给队列。

![image-20211009145036134](D:\1\blog\source\_posts\kafka\image-20211009145036134.png)

topic只是一种中转的模式。

## 3. 消息

#### 3.1 单播消息

在一个kafka的topic中，启动两个消费者，一个生产者。这条消息是否会被两个消费者消费？

如果多个消费者在同一个消费组，那么只有一个消费者可以收到订阅的topic中的消息。换言之，同一个消费组中只能有一个消费者收到一个topic的消息。

#### 3.2 多播消息

不同的消费组订阅同一个topic，那么不同的消费组中只有一个消费者能够收到消息。

通过以下命令查看消费组的信息：

```
./kafka-consumer-groups.sh --bootstrap-server 172.16.253.38:9092 --describe --group testGroup
```

## 4.基本概念

 ![image-20211009210848040](D:\1\blog\source\_posts\kafka\image-20211009210848040.png)

一个主题中的消息量是非常大的，因此可以通过分区的设置，来分布式存储这些消息。比如一个topic创建了3个分区，那么topic中的消息就会被分别存放在3个分区中。

一个主题创建多个分区：

```
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partition 2 --topic testGroups
```

#### 4.1 kafka中消息日志文件中保存的内容

* 00000.log：这个文件中保存的就是消息。

* _consumer_offset-49: kafka内部创建了_consumer_offset主题包含了50个分区。这个主题用来存放消费者消费某个主题的偏移量。

  * 消费者定期将自己消费的offset提交给kafka内部topic：_consumer_offsets，提交过去的时候，key是consumerGroupId+topic + 分区号，value就是当前offset的值，kafka会定期清理topic里的消息，组后保留最新的那条数据。

  * 因为_consumer_offsets可能会接收高并发的请求，kafka默认给其分配50个分区（可以通过offset.topic.num.partitions设置），这样可以通过加机器的方式抗住大并发。

  * 通过如下公式选出consumer消费的offset要提交到_consumer_offset的哪个分区：

    公式：hash(consumerGoupId)%_consumer_offsets主题的分区数

  #### 4.2 副本的概念
  
  副本是对分区的备份。在集群中，不同的副本会被部署在不同的broker上。下面的例子：创建1个主题，2个分区，3个副本。
  
  ```bash
  ./kafka-topic.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 2 --topic my-replicated-topic
  ```
  
  ![image-20211010110637908](D:\1\blog\source\_posts\kafka\image-20211010110637908.png)
  
  * isr：
  
    可以同步的broker节点和已经同步的broker节点，放在isr集合中。

总结：集群中有多个broker，创建主题时可以指明主题有多个分区（把消息拆分到不同的分区中存储），可以为分区创建多个副本，不同的副本存放在不同的broker里。

## 5. Kafka集群消息的消费

生产者生产消息：

```bash
./kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --topic my-replicated-topic
```

生产者消费消息：

```bash
./kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --from-beginning --topic my-replicated-topic
```

分区分消费组的集群消息中的细节：

![image-20211010113414524](D:\1\blog\source\_posts\kafka\image-20211010113414524.png)

如果消费者挂了，会触发rebalance机制，让其他消费者来消费该分区。

#### 5.1 同步发送消息

![image-20211010202240336](D:\1\blog\source\_posts\kafka\image-20211010202240336.png)

#### 5.2 异步发送消息

异步就是有了一个回调，但是异步方式会丢失数据。平常一般都是会使用同步方式进行发送。

![image-20211010203700110](D:\1\blog\source\_posts\kafka\image-20211010203700110.png)

* kafka默认会创建一个消息缓冲区，用来存放要发送的数据，缓冲区是32m

```java
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
```

* kafka本地线程会去缓冲区一次拉16k的数据，发送到broker

```java
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
```

#### 5.3 提交offset

* 自动提交

![image-20211010211840464](D:\1\blog\source\_posts\kafka\image-20211010211840464.png)

pull消息，pull消息下来之后，立马直接提交offset。

* 手动提交

你消费时或者消费后再提交offset

![image-20211010212041257](D:\1\blog\source\_posts\kafka\image-20211010212041257.png)

* 手动提交offset提交参数

```java
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
```

* 在消费完消息后进行手动提交

  * 手动同步提交

    ```java
    if (records.count() > 0) {
        // 手动同步提交offset，当前线程会阻塞直到offset提交成功
        // 一般使用同步提交，因为提交后一般没有什么逻辑代码了
        consuemr.commitSync();
    }
    ```

  * 手动异步提交

    ```java
    ```

* 提交的内容

消费者无论是自动提交还是手动提交，都需要把所属的消费组 + 消费的某个主题 + 消费的某个分区及消费的偏移量，这样的信息提交到集群的主题里的_consumer_offsets主题里面。

#### 5.4 消费者poll消息

* 消费者建立了与broker之间的长连接，开始poll消息。
* 默认一次poll500条消息

```java
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);
```

可以根据消费速度的快慢来设置，因为如果两次poll的时间超出了30s的时间价格，kafka会认为其消费能力过弱，将其提出消费组。将分区分配给其他消费者。

可以通过这个值进行配置：

```java
props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30 * 1000);
```

* 如果1s内每隔1s内没有poll到任何消息，则继续去poll消息，循环往复，直到poll到消息。如果超出了1s，则此次长轮询结束。

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
```

* 消费者发送心跳的间隔

```java
props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000);
```

* kafka如果超过10秒没有收到消费者的心跳，则会把消费提出消费组，进行rebalance，把分区分配给其他消费者。

```java
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000);
```

#### 5.5 指定分区和偏移量、时间消费

* 指定分区消费

```java
consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
```

* 从头消费

```java
consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0));
consumer.seekToBeginning(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
```

* 指定offset消费

```java
consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
consumer.seek(new TopicPartition(TOPIC_NAME, 0), 10);
```

* 指定时间来消费

根据时间，去所有的partition中确定该事件对应的offset，然后去所有的partition中找到该offset的之后的消息开始消费。

```java
```



#### 5.6 新消费组的偏移量

如果设置了"earlier"。找这个消费组最近一次消费的offset位置。

如果没有设置，那么这个新的消费组只会收到新发的加入时间之后发送的消息。

## 6. springboot 中使用kafka

#### 6.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

#### 6.2 编写配置文件

```yaml
server:
  port: 8080

spring:
  kafka:
    bootstrap-servers: 172.16.2.11:9092
    producer:
      retries: 3 # 设置大于0的值，则客户端会将发送失败的记录重新发送
      batch-size: 16384
      buffer-memory: 33554432
      acks: 1
      # 指定消息key和消息体的编解码方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: default-group
      enable-auto-commit: false
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      max-poll-records: 500
    listener:
      ack-mode: MANUAL_IMMEDIATE
    # 每当一条记录被消费者监听处理之后提交
      # RECORD
      # 当每一批poll（）的数据被消费者监听器（ListenerConsumer)处理之后提交
      # BATCH
      # 每当一批poll（）的数据被消费者监听器(ListenerConsumer)处理之后，被处理的record数量大于COUNT时提交
      # COUNT
      # TIME | COUNT 有一个条件满足时提交
      # 手动调用Acknowledge.acknowledge()后立即提交，一般使用这种
      # MANUAL_IMMEDIATE

```

#### 6.3 消息生产者

```java
@RestController
@RequestMapping("/msg")
public class MyKafkaController {

    private final String TOPIC_NAME = "my-replicated-topic";

    private KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    public void setKafkaTemplate(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @RequestMapping("/send")
    public String sendMessage() {
        kafkaTemplate.send(TOPIC_NAME, 0, "key", "This is a message!");
        return "send success!";
    }

}

```

#### 6.4 编写消费者

```java
@Component
public class MyConsumer {

    @KafkaListener(topics = "my-replicated-topic", groupId = "MyGroup1")
    public void listenGroup(ConsumerRecord<String, String> record, Acknowledgment ack) {
        String value = record.value();
        System.out.println(value);
        System.out.println(record);
        // 手动提交offset
        ack.acknowledge();
    }
    
}
```

## 7. Kafka集群Controller、Rebalance和HW

### 7.1 Controller

为什么？

一个分区有多个副本，这些副本里面有一个leader。但是当leader挂了，谁来选举呢？所以需要controller。  

kafka集群中的broker在zk中创建临时序号节点，序号最小的节点（最先创建的节点）将作为集群的controller，负责管理整个集群中的所有分区和副本的状态。

* 当某个分区的leader副本出现故障时，由控制器负责为该分区选举新的leader副本。选举的规则是从isr集合中最左边获得。
* 当检测到某个分区的ISR集合发生变化时，由控制器负责通知所有broker更新其元数据信息。
* 当使用kafka-topic.sh脚本为某个topic增加分区数量时，同样还是由控制器负责让新分区被其他节点感知到。

### 7.2 rebalance机制

![image-20211011165524387](D:\1\blog\source\_posts\kafka\image-20211011165524387.png)

消费者挂了或者消费者新加入，都会出发rebalance机制。本质上是由于消费者没有指明分区消费或者分区没有被任何消费者消费。

这个机制会重新调整消费者消费哪个分区。

出发rebalance分区策略：

* range 通过公式计算某个消费者消费哪个分区。（分区总数/n （n是消费者个数，但比如10/3 = 3 余 1 ，除不尽，那么 消费者 C1-0 （第一个消费者）便会多消费 1 个分区：）)
  * 弊端：如上，只是针对 1 个 topic 而言，C1-0消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1-0 都将多消费 1 个分区，topic越多，C1-0 消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了。
* 轮询：大家轮着消费。 触发之后需要重新分配，耗费性能。
* sticky：触发了rebalance后，在消费者消费的原分区不变的基础上进行调整。

### 7.3 HW

HW俗称高水位，HighWatermark的缩写，取一个partition对应的ISR中最小的LEO（log-end-offset)作为HW，consumer最多只能消费到HW所在的位置。另外每个replica都有HW，leader和follwer各自负责更新自己的HW的状态。对于leader新写入的消息，consumer不能立刻消费，leader会等待该消息被所有ISR中的replicas同步后更新HW，此时消息才能被consumer消费。这样就保证了如果leader所在的broker失效，该消息仍然可以从新选举的leader中获取。

## 8. 优化

### 8.1 如何防止消息丢失

* 发送方：ack是1或者-1/all 可以防止消息丢失，如果要做到99.9999%，ack设置为all，把min.insync.replicas配置成分区备份数
* 消费方：把自动提交改为手动提交



### 8.2 如何防止消息重复消费

如果把生产端的重试机制关闭、消费端的手动提交改成自动提交，这样反而会消息丢失。那么可以在防止消息丢失时加上消费消息的幂等性保证。

幂等性保证：

* mysql插入业务id为主键，主键是唯一的，所以一次只能插入一条。
* 使用redis或zk分布式锁（主流的方案）

![image-20211011185818080](D:\1\blog\source\_posts\kafka\image-20211011185818080.png)



### 8.3 如何做到顺序消费

* 发送方： 使用同步发送，等到发送成功再发送下一条，确保消息是顺序发送的。
* 接收方：消息是发送一个分区中，只能有一个消费组的消费者来接收消息。

因此，kafka的顺序消费会牺牲性能。

### 8.4 消息积压问题

这个问题会产生：磁盘满了、生产端发消息导致kafka性能过慢，容易出现服务雪崩（随着被消费的数据堆积越来越多，消费者寻址的能力越来越差，最后导致整个kafka对外提供服务的性能很差，从而造成其他服务也访问很慢，需要有：

* 方案一：在一个消费者中启动多个线程，让多个线程同时消费。---提升一个消费者的消费能力
* 方案二： 如果方案一不够，这个时候启动多个消费者，多个消费者部署在不同的服务器上。其实多个消费者部署在同一个服务器上也可以提高消费能力——充分利用服务器的CPU资源。
* 方案三：让一个消费者去把收到的消息往另外一个topic上发，另外一个topic设置多个分区和多个消费者，进行具体的业务消费。

### 8.5 延迟队列

应用场景：在订单创建后如果超过30分钟没有付款，则需要取消订单，此时可用延时队列来实现：

* 创建多个topic，每个topic表示延时的间隔
  * topic_5s：延时5s执行的队列
  * topic_1m: 延时1分钟执行的队列
  * topic_30m:延时30分钟执行的队列
* 消息发送者发送消息到相应的topic，并带上消息发送的时间
* 消息订阅相应的topic，消费时轮询消费整个topic中的消息
  * 如果消息发送时间，和消费的当前时间超过预设的值，比如30分钟
  * 如果消息发送时间，和消费的当前时间没有超过预设的值，则不消费当前的offset及之后offset的所有消息都消费
  * 下次继续消费该offset处的消息，判断时间是否已经满足预设值



## 9.Kafka-eagle监控平台

kafka eagle 官网：[Preface - Kafka Eagle (kafka-eagle.org)](https://docs.kafka-eagle.org/)

* 安装
  * jdk必须安装

* 解压安装包
* 修改配置文件 `vim system-config.properties` 修改zk地址和mysql地址
* 



























