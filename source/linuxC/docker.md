### docker

#### 宿主机启动

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
#宿主机zookeeper启动
sh /home/dong/win_share_file/zookeeper/bin/zkServer.sh start
#宿主机kafka启动
sh /home/dong/win_share_file/kafka/bin/kafka-server-start.sh -daemon /home/dong/win_share_file/kafka/config/server.properties
#节点启动
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

# 方法 1：通过 kafka-server-start.sh 查看
/home/dong/win_share_file/kafka/bin/kafka-server-start.sh --version

# 方法 2：通过 kafka-topics.sh 查看
/home/dong/win_share_file/kafka/bin/kafka-topics.sh --version
```

####  docker机启动

```
# 安装 Docker
sudo dnf install docker
sudo systemctl start docker
sudo systemctl enable docker

# 创建 3 个 AlmaLinux 容器
sudo docker run -d --name container1 -it almalinux:9 /bin/bash
sudo docker run -d --name container2 -it almalinux:9 /bin/bash
sudo docker run -d --name container3 -it almalinux:9 /bin/bash

# 管理容器
sudo docker ps
sudo docker exec -it container1 bash
```

启容器，因是podman的，不需要启动docker服务

```
# 查看现有容器
sudo docker ps -a

# 如果容器已存在，直接启动
sudo docker start server1
sudo docker start server2
sudo docker start server3

# 查看运行状态
sudo docker ps

#进入容器
sudo docker exec -it server1 bash
sudo docker exec -it server2 bash
sudo docker exec -it server3 bash

#启动kafka
sh /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
```

rz命令的安装包

```
yum install -y lrzsz
```

netstat命令安装包

```
yum install -y net-tools
```

ss命令安装

```
yum install -y iproute
```

ls命令安装

```
yum install -y coreutils
```

ssh服务安装

```
# 安装 SSH 服务
yum install -y openssh-server openssh-clients
# 生成 SSH 密钥
ssh-keygen -A
# 启动 SSH 服务
/usr/sbin/sshd
# 在宿主机执行
ssh-keygen -R 10.88.0.2
```

almaLinux使用的是podman docker

这些是能拉取成功的

```
sudo docker run -d --name server1 quay.io/centos/centos:stream9 sleep infinity
sudo docker run -d --name server2 quay.io/centos/centos:stream9 sleep infinity
sudo docker run -d --name server3 quay.io/centos/centos:stream9 sleep infinity
```

查看docker容器ip

```shell
sudo docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress}}' server1
```

这是创建成功的

```shell
# 删除旧容器
sudo docker rm -f server1

# 使用 --privileged 创建
sudo docker run -d --privileged --name server1 quay.io/centos/centos:stream9 sleep infinity

#进入容器
sudo docker exec -it server1 bash
sudo docker exec -it server2 bash
sudo docker exec -it server3 bash

# 配置 SSH
sudo docker exec -it server1 bash -c "
yum install -y openssh-server && \
ssh-keygen -A && \
echo 'root:1' | chpasswd && \
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config && \
echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config && \
/usr/sbin/sshd
"
```

安装ps命令

```shell
yum install -y procps
```

### zookeeper安装启动：

```
tar -xzf apache-zookeeper-3.8.6-bin.tar.gz
mv apache-zookeeper-3.8.6-bin zookeeper

# 创建数据目录和 myid
mkdir -p /opt/zookeeper/data
echo "1" > /opt/zookeeper/data/myid

# 创建配置文件
cat > /opt/zookeeper/conf/zoo.cfg << 'EOF'
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/zookeeper/data
clientPort=2181
admin.enableServer=false
EOF

# 启动 ZooKeeper
/opt/zookeeper/bin/zkServer.sh start

# 检查状态
/opt/zookeeper/bin/zkServer.sh status
```

### kafka安装启动

jdk要安装17版本的

```
yum install -y java-17-openjdk java-17-openjdk-devel
# 检查 Java 版本
java -version
# 设置 Java 17 为默认
alternatives --config java
# 选择 java-17 的编号

#在 /usr/lib/jvm 目录下将，java版本目录改成 java
export JAVA_HOME=/usr/lib/jvm/java
export PATH=$JAVA_HOME/bin:$PATH
```

启动kafka，版本是要2.8.1的版本

```
tar -xzf kafka_2.13-2.8.1.tgz
mv kafka_2.13-2.8.1 kafka
# 创建日志目录
mkdir -p /opt/kafka/logs

# 配置 Kafka
cat > /opt/kafka/config/server.properties << 'EOF'
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://docker镜像的ip:9092
log.dirs=/opt/kafka/logs
num.partitions=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
zookeeper.connect=docker镜像的ip:2181
EOF

# 启动 Kafka
/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties

# 检查状态
ps aux | grep kafka
ss -tlnp | grep 9092
```



--docker kafka启动

```
sh /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
```

在server1、2、3中安装librdkafka

```
yum install -y epel-release
yum install -y librdkafka-devel
```

拷贝服务

```
sudo docker cp ka_producer server1:/home/bin/
sudo docker cp ka_producer server2:/home/bin/
sudo docker cp ka_producer server3:/home/bin/
```

#### kafka启不起来的问题

```
[2026-04-20 14:36:09,461] ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.common.InconsistentClusterIdException: The Cluster ID 3zeUWQl1SxKjwBcostx3cw doesn't match stored clusterId Some(2KzpIlJPRveCnclTCpXOOg) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.
```

解决办法：删除 rm /opt/kafka/logs/meta.properties



#### topic重建

```
# 先删除旧 topic
sh kafka-topics.sh --bootstrap-server 192.168.171.128:9092 --delete --topic dongThing

# 重新创建，设置合适的副本因子
sh kafka-topics.sh --bootstrap-server 192.168.171.128:9092 --create --topic dongThing --partitions 3 --replication-factor 1
```

### 开放端口

```
sudo firewall-cmd --add-port=2181/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

