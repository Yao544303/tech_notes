# Kafka常用命令

## 启动kafka

```
bin/kafka-server-start.sh config/server.properties
```

## 创建topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181--replication-factor 1 --partitions 1 --topic test
```

## 查询topic 列表和topic 描述信息

```
bin/kafka-topics.sh --list --zookeeper localhost:2181 Test
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
```

## 向topic 发送测试消息

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

## 读取topic 里的消息

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
This is a message
This is another message
```
