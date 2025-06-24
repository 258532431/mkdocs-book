---
comments: true
---

## 本地`Docker`部署单节点`RocketMQ`

### 部署MQ服务
```bash
# 拉取镜像
docker pull apache/rocketmq:5.3.2

# 创建容器共享网络
docker network create rocketmq

# 启动NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.2 sh mqnamesrv
# 或
docker run -d --name rmqnamesrv --network rocketmq -p 9876:9876 \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=0.0.0.0:9876" apache/rocketmq:5.3.2 sh mqnamesrv

# 验证 NameServer 是否启动成功
docker logs -f rmqnamesrv
```

### 配置broker
```bash
# 本地创建 broker.conf 文件，namesrvAddr使用容器名代替 IP 地址，但宿主机无法发送消息，可添加 brokerIP1=宿主机IP
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
namesrvAddr = rmqnamesrv:9876
brokerIP1=192.168.132.95
```

### 启动broker
```bash
# docker配置内存不足可能导致启动失败，需要2G-4G内存
docker run -d \
  --name rmqbroker \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=rmqnamesrv:9876" \
  -p 10911:10911 -p 10909:10909 \
  --network rocketmq \
  -v /Users/ycl/Desktop/soft/docker/conf/broker.conf:/etc/rocketmq/broker.conf \
  apache/rocketmq:5.3.2 \
  sh mqbroker -c /etc/rocketmq/broker.conf
```

## 部署MQ Dashborad

```bash
# 拉取镜像
docker pull apacherocketmq/rocketmq-dashboard:latest

# 如果mq服务也部署在docker中，配置 Drocketmq.namesrv.addr=mq服务容器名:9876， 如果mq服务部署在宿主机，则配置 
docker run -d --name rocketmq-dashboard --network rocketmq \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=rmqnamesrv:9876" \
  -p 8080:8080 \
  -t apacherocketmq/rocketmq-dashboard:latest
```
