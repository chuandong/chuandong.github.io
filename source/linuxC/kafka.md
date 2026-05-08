### kafka

启动zookeeper：

```shell
cd /home/dong/win_share_file/kafka
nohup bin/zookeeper-server-start.sh config/zookeeper.properties > /tmp/zookeeper.log 2>&1 &
```

启动kafka：

```shell
cd /home/dong/win_share_file/kafka
nohup bin/kafka-server-start.sh config/server.properties > /tmp/kafka.log 2>&1 &

#停kafka
pkill -f kafka.Kafka

/opt/zookeeper/bin/zkServer.sh start

/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
```

验证kafka服务：

```
# 查看主题列表
cd /home/dong/win_share_file/kafka && bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# 测试消息生产
cd /home/dong/win_share_file/kafCode && ./ka_producer

# 测试消息消费
cd /home/dong/win_share_file/kafCode && ./ka_consumer

--查询topic
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

