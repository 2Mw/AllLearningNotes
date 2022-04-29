# RocketMQ

[TOC]

[BV1L4411y7mn](https://www.bilibili.com/video/BV1L4411y7mn?p=29) P29

MQ 消息队列可以进行限流削峰，异步解耦，数据收集。

下载链接：[Link](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.9.3/rocketmq-all-4.9.3-bin-release.zip)

## 一. RocketMQ 基础

### 1. 初步

启动RocketMQ：

1. 启动 Name Server

   ```sh
   nohup sh bin/mqnamesrv &
   tail -f ~/logs/rocketmqlogs/namesrv.log
   ```

2. 启动 Broker 连接 name server

   ```sh
   nohup sh bin/mqbroker -n localhost:9876 &
   tail -f ~/logs/rocketmqlogs/broker.log
   ```

可以通过 `jps` 来查看是否启动成功。如果未启动成功可以是由于设备硬件原因导致，可以通过在 RocketMQ 的bin 目录下修改 `runbroker.sh` 以及 `runserver.sh` 中的 JVM 中配置进行修改，其中 JVM 中内存的设置默认为：

```
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g"
```

关闭 name server 和 broker：

```sh
sh bin/mqshutdown namesrv
sh bin/mqshutdown broker
```

测试发送消息和接受消息：

1. 首先需要对接收方和发送方都需要配置环境变量

   ```sh
   export NAMESRV_ADDR=localhost:9876
   ```

2. 使用 RocketMQ 中的 demo 进行发送和接收消息

   ```sh
   # 发送消息
   sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
   # 接收消息
   sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
   ```

### 2. RocketMQ 介绍

![image-20220428105803751](rocketmq.assets/image-20220428105803751.png)

RocketMQ 中角色介绍：

1. NameServer：NameServer 是 Broker 的管理者，Broker 需要主动上报自己的信息到 NameServer
2. Broker：用于暂存和传输消息。Broker Master主要负责写操作，Slave 主要负责写操作。
3. Producer：主要职责是发送消息，首先询问 NameServer 找到对应的 Broker，然后向 Broker 发送消息。
4. Consumer：主要职责是接收消息，询问 NameServer 找到对应的 Broker，然后向 Broker 拿去消息。
5. Topic：消息的种类。
6. Message Queue：相当于是 Topic 的分区，用于并行消息发送和消息处理。

RocketMQ 集群特点：

1. NameServer 是几乎无状态的节点，可以集群部署，节点之间无任何信息同步。
2. Broker 之间分为 Master 和 Slave 节点，一个 Master 可以对应多个 Slave 节点，但一个 Slave 只能有一个 Master。BrokerId 中 0 表示 Master，其他表示 Slave。
3. Producer 和 NameServer 集群中其中一个节点建立长连接，从 NameServer 中获取 Topic 的路由信息

RocketMQ 集群搭建方式：

由以上可知，RocketMQ 集群搭建的核心就是对 Broker 集群的搭建，对 Broker 集群搭建的方式分为以下几种：

1. 单 Master 模式：不可靠，不应该叫做集群。
2. 多 Master 模式：集群无 Slave 全是 Master，配置简单
3. 多 Master 多 Slave (异步)：即 Master 和 Slave 之间同步数据的方式是异步的。
4. 多 Master 多 Slave (同步)：性能会比异步的方式要低一些。

### 3. RocketMQ 集群搭建

集群搭建 Demo：

| 序号 | ip            | 角色               | 架构模式        |
| ---- | ------------- | ------------------ | --------------- |
| 1    | 192.168.1.100 | nameserver, broker | Master1, Slave2 |
| 2    | 192.168.1.103 | nameserver, broker | Master2, Slave1 |

1. 配置网络信息

   ```sh
   vim /etc/hosts
   ```

   添加以下配置：

   ```
   # rocketmq address
   # === name server
   192.168.1.100 rmq-ns1
   192.168.1.103 rmq-ns2
   # === master / slave
   192.168.1.100 rmq-m1
   192.168.1.100 rmq-s2
   192.168.1.103 rmq-m2
   192.168.1.103 rmq-s1
   ```

   重启网卡：

   ```sh
   systemctl restart network	# centos
   systemctl restart NetworkManager # ubuntu
   ```

   测试域名是否正确配置：

   ```sh
   ping rmq-ns1
   ping rmq-ns2
   ```

2. 关闭防火墙

   rocketmq 常用的端口有：9876(ns)，10911(master)，11011(slave)

   ```sh
   # 直接禁用
   systemctl stop firewall-cmd
   firewall-cmd --state
   systemctl disable firewall-cmd
   # 或者选择开放某些端口
   firewall-cmd --add-port=9876/tcp --permenant
   firewall-cmd --add-port=10911/tcp --permenant
   firewall-cmd --add-port=11011/tcp --permenant
   firewall-cmd --reload
   ```

3. 配置环境变量：

   ```sh
   vim /etc/profile
   ```

   添加以下内容：

   ```
   ROCKETMQ_HOME=/opt/rocketmq-4.9.3
   PATH=$PATH:$ROCKETMQ_HOME/bin
   export PATH
   ```

   使环境变量生效：

   ```sh
   source /etc/profile
   ```

4. 创建消息存储路径（消息持久化存储的位置）

   ```sh
   mkdir /usr/local/rocketmq/store
   cd /usr/local/rocketmq/store
   mkdir commitlog consumequeue index
   ```

5. Broker 文件配置

   Master 配置：

   ```
   # cluster name
   brokerClusterName=rmq-cluster
   brokerName=broker-a
   # 0 -> Master, other -> Slave
   brokerId=0
   # Name server address
   namesrvAddr=rmq-ns1:9876;rmq-ns2:9876
   # create the topic which the server doesn't exists, the default number of queue.
   defaultTopicQueueNums=4
   # 是否允许broker自动创建topic，建议线下开启，向上关闭
   autoCreateTopicEnable=true
   # 是否允许broker自动创建订阅组，建议线下开启，线上关闭
   autoCreateSubscriptionGroup=true
   # Broker 对外监听的端口
   listenPort=10911
   # 删除文件的时间点-04:00
   deleteWhen=04
   # 文件保留时间，默认48小时，这里是120小时
   fileReservedTime=120
   # commitLog 每个文件的默认大小，默认为1G
   mapedFileSizeCommitLog=1073741824
   # ConsumeQueue 每个文件默认存储的数据数目，默认 30w
   mapedFileSizeConsumeQueue=300000
   # 检测物理文件磁盘空间
   diskMaxUsedSpaceRatio=88
   # 消息持久化的路径
   storePathRootDir=/usr/local/rocketmq/store
   # commitLog 持久化的路径
   storePathCommitLog=/usr/local/rocketmq/store/commitlog
   # 消息队列持久化的路径
   storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
   # 消息索引存储路径
   storePathIndex=/usr/local/rocketmq/store/index
   # checkpoint 文件存储路径
   storePathCheckpoint=/usr/local/rocketmq/store/checkpoint
   # abort 文件存储路径
   storePathAbort=/usr/local/rocketmq/store/abort
   # 限制消息的大小
   maxMessageSize=65536
   
   # Broker 的角色
   # ASYNC_MASTER / SYNC_MASTER / SLAVE
   brokerRole=SYNC_MASTER
   # 刷盘的方式 SYNC_FLUSH  ASYNC_FLUSH
   flushDiskType=SYNC_FLUSH
   ```

   对于 Slave 的配置只需要修改 brokerId，brokerRole，listenPort以及flushDiskType，Master 刷盘的方式为同步刷盘，Slave 的刷盘方式为异步刷盘。不同的 Master 之间记得修改名称。

   **注意**：同一主机下的不同的broker持久化的路径也需要进行改变，比如 store1，store2。

6. 启动 NameServer 和 Broker

   ```sh
   # 启动 nameserver
   nohup sh mqnamesrv &
   # 启动 master1 和 slave2
   nohup sh mqbroker -c /opt/rocketmq-4.9.3/conf/2m-2s-sync/broker-a.properties &
   nohup sh mqbroker -c /opt/rocketmq-4.9.3/conf/2m-2s-sync/broker-b-s.properties &
   # 启动另一个主机的 master2 和 slave1
   nohup sh mqbroker -c /opt/rocketmq-4.9.3/conf/2m-2s-sync/broker-b.properties &
   nohup sh mqbroker -c /opt/rocketmq-4.9.3/conf/2m-2s-sync/broker-a-s.properties &
   ```

   查看是否启动成功：

   ```sh
   [root@localhost rocketmq-4.9.3]# jps
   11028 BrokerStartup
   11015 BrokerStartup
   11016 NamesrvStartup
   11197 Jps
   ```

### 4. MQAdmin 的使用

也可以直接使用dashboard

### 5. 集群监控平台

链接：[apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)

```sh
git clone https://github.com/apache/rocketmq-dashboard.git
# 进入到dashboard目录，进行编译
mvn clean package -Dmaven.test.skip=true
java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```

在mvn编译之前需要到源代码中的 `application.yml` 中添加 namesrv 的地址。当然不添加也可以，在进入网页中的配置也行。

然后进入网页 [localhost:8080](http://localhost:8080/#/) 即可访问查看 dashboard。

## 二. RocketMQ 消息发送

首先需要导入maven依赖：

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.9.3</version>
</dependency>
```

生产者消息发送步骤：

1. 创建消息生产者Producer，并且制定生产者组名
2. 指定 NameServer 地址
3. 启动 Producer
4. 创建消息对象，指定主题 Topic，Tag和消息体
5. 发送消息
6. 关闭生产者 Producer

消息消费者步骤：

1. 创建消费者 Consumer，指定消费者组名
2. 指定 NameServer 地址
3. 订阅主题 Topic 和 Tag
4. 设置回调函数处理消息
5. 启动消费者 Consumer

### 1. 基本demo

🔵生产者发送消息：

1. 发送同步消息：

   ```java
   public class SyncProducer {
       // 发送同步消息
       public static void main(String[] args) throws Exception {
           DefaultMQProducer producer = new DefaultMQProducer("group1");
           producer.setNamesrvAddr(Config.getNameServersString());
           producer.start();
   
           for (int i = 0; i < 10; i++) {
               // 指定 topic， tag， 消息内容
               Message msg = new Message("base", "tag1", ("Hello world_" + i).getBytes(StandardCharsets.UTF_8));
               SendResult result = producer.send(msg);
               System.out.println(result);
               TimeUnit.SECONDS.sleep(1);
           }
   
           producer.shutdown();
       }
   }
   ```

2. 发送异步消息

   发送同步和异步消息之间的区别就是在 `send` 函数中添加回调函数即可。

   ```java
   public class AsyncProducer {
       public static void main(String[] args) throws Exception {
           DefaultMQProducer producer = new DefaultMQProducer("group1");
           producer.setNamesrvAddr(Config.getNameServersString());
           producer.start();
           for (int i = 0; i < 10; i++) {
               Message msg = new Message("base", "tag2", ("[Async] Hello world_" + i).getBytes(StandardCharsets.UTF_8));
               producer.send(msg, new SendCallback() {
                   @Override
                   public void onSuccess(SendResult sendResult) {
                       System.out.println("发送成功 " + sendResult);
                   }
   
                   @Override
                   public void onException(Throwable throwable) {
                       System.out.println("发送失败，出现错误: " + throwable);
                   }
               });
   
               TimeUnit.SECONDS.sleep(1);
           }
           producer.shutdown();
       }
   }
   ```

3. 发送单向消息

   发送单向消息的流程和之前一模一样，只不过发送的时候使用的函数时 `sendOneway()`，无需结果的接收，比如写入日志等操作。

🔵消费者接收消息

消费者的模式可以分为广播模式和负载均衡模式，默认为负载均衡模式。

对于每个消息都被消费者消费一遍就是广播模式；一个消息只能被一个消费者消费就是负载均衡模式。

1. 负载均衡模式：

   ```java
   public class BaseConsumer {
       public static void main(String[] args) throws Exception {
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
           consumer.setNamesrvAddr(Config.getNameServersString());
           consumer.subscribe("base", "tag1");
           consumer.registerMessageListener((MessageListenerConcurrently) (msgs, ctx) -> {
               msgs.forEach(i -> {
                   System.out.println(new String(i.getBody()));
               });
               return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
           });
           consumer.start();
       }
   }
   ```

2. 广播模式：

   需要设置消息模式：`consumer.setMessageModel(MessageModel.BROADCASTING)`

   ```java
       public static void main(String[] args) throws Exception {
           DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
           consumer.setNamesrvAddr(Config.getNameServersString());
           consumer.subscribe("base", "tag1");
           consumer.setMessageModel(MessageModel.BROADCASTING);
           consumer.registerMessageListener((MessageListenerConcurrently) (msgs, ctx) -> {
               msgs.forEach(i -> {
                   System.out.println(new String(i.getBody()));
               });
               return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
           });
           consumer.start();
       }
   ```

### 2. 顺序消息

参考：[Order Message](https://rocketmq.apache.org/docs/order-example/)

保证消息的发送和消费顺序是一致的。

![image-20220429150054360](rocketmq.assets/image-20220429150054360.png)

一个 Broker 中含有多个消息队列，生产者和消费者都是采用多线程的方式从消息队列中生产消息和消费消息。

可以采用局部有序的方式来保证对某个业务顺序的保证，为某个业务分配特定的消息队列。

生产者：

```java
public class OrderedProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("g1");
        producer.setNamesrvAddr(Config.getNameServersString());
        producer.start();
        String[] tags = {"TagA", "TagB", "TagC", "TagD", "TagE"};
        for (int i = 0; i < 100; i++) {
            int orderId = i % 10;
            Message msg = new Message("OrderedTopic", "TagA", "KEY_" + i, ("Hello RocketMQ_" + orderId + "_" + i).getBytes(StandardCharsets.UTF_8));
            SendResult res = producer.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                    Integer orderId = (Integer) o;
                    int index = orderId % list.size();
                    // 返回选取的 MQ
                    return list.get(index);
                }
            }, orderId);

            System.out.println(res);
        }
        producer.shutdown();
    }
}
```

生产者根据某些特定的符号选择对应的消息队列。

消费者使用 `MessageListenerOrderly` 类来进行消费，这个类会为对每个消息队列分配单独的消费线程。

```java
@Slf4j
public class OrderedConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("g1");
        consumer.setNamesrvAddr(Config.getNameServersString());
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        consumer.subscribe("OrderedTopic", "TagA || TagC || TagD");
        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext ctx) {
                list.forEach(i -> {
                    log.debug("Recv msg: {}", new String(i.getBody()));
                });
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        consumer.start();
    }
}
```

### 3. 定时消息

当生产者生产一个消息的时候，并不会被立即消费，而是等待特定时间后再去消费。在RocketMQ 中消息的级别分为18个级别。

![image-20220429163027713](rocketmq.assets/image-20220429163027713.png)

只需要设置 message 的 `setDelayTimeLevel` 即可。

```java
public static void main(String[] args) throws Exception {
    //  省略 ...
    msg.setDelayTimeLevel(3);
    //  省略 ...
    }
    producer.shutdown();
}
```

### 4. 发送批量消息

即将多个 Message 放到 List 中。

当然发送批量消息的时候还需要注意，发送消息的大小超过4MB的时候，最好将消息进行分割。

可以参考官网的代码：[Batch Example - Apache RocketMQ](https://rocketmq.apache.org/docs/batch-example/)







