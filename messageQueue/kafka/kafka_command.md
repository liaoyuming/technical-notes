# kafka 命令
kafka 版本为 kafka_2.11-1.0.1

官方文档地址: [kafka](https://kafka.apache.org/)

## 启动
#### 启动 zookeeper
```
# 安装 brew install zookeeper
zkServer start
```
#### 启动 kafka
```
./bin/kafka-server-start.sh config/server.properties
```

## 生产
#### 创建 topic
config/server.properties 里配置自动创建 topic

```
auto.create.topics.enable=true
```
手动创建

```
# replication-factor 副本
# partitions 分区
./bin/kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic topic_name
```
#### 生产消息
```
./bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic topic_name
```

## 消费
#### 消费数据
```
./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic topic_name --from-beginning

./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test --consumer-property group.id=dev --from-beginning
```

## 查询
#### 查看 topic 列表
```
./bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list
```
#### 查看 topic 状态

```
./bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic topic_name --describe 
```
#### 查看 消费者列表
```
./bin/kafka-consumer-groups.sh --zookeeper 127.0.0.1:2181 --list
```
#### 查看 消费者状态
```
./bin/kafka-consumer-groups.sh --zookeeper 127.0.0.1:2181  --group consumer_name --describe
```
## 删除
#### 删除 topic
```
./bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --delete --topic topic_name
```
