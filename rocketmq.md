# RocketMQ

[TOC]

[BV1L4411y7mn](https://www.bilibili.com/video/BV1L4411y7mn?p=7) P7

MQ 消息队列可以进行限流削峰，异步解耦，数据收集。

下载链接：[Link](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.9.3/rocketmq-all-4.9.3-bin-release.zip)

## 一 RocketMQ 基础

### 1. 初步

启动RocketMQ：

1. 启动 Name Server

   ```sh
   nohup sh bin/mqnamesrv &
   tail -f ~/logs/rocketmqlogs/namesrv.log
   ```

2. 启动 Broker

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

