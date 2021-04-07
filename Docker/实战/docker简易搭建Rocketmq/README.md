
# docker 简易搭建Rocketmq

#1.启动NameServer：
```
docker run -d -p 9876:9876 --name rmqserver  foxiswho/rocketmq:server-4.5.1
```

#2.启动broker：
```
docker run -d -p 10911:10911 -p 10909:10909\
 --name rmqbroker --link rmqserver:namesrv\
 -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt"\
 -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m"\
 -v /home/rocketmq/conf/broker.conf:/etc/rocketmq/broker.conf \
 foxiswho/rocketmq:broker-4.5.1;
```
注：使用自身配置文件 -v /conf/broker.conf:/etc/rocketmq/broker.conf ,参考附录brokerIP1 配置上宿主机的IP（当部署在外网上 看到console集群中地址是否是外网IP|Caused by: org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout 无法发送成功）

#3.启动console:
```
docker run -d --name rmqconsole -p 8080:8080 --link rmqserver:namesrv\
 -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876\
 -Dcom.rocketmq.sendMessageWithVIPChannel=false"\
 -t styletang/rocketmq-console-ng;
```

#4.springboot添加依赖
```
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.0.4</version>
        </dependency>
```
rocketmq对应的依赖版本对应软件版本
```
2.0.2对应rocketmq 4.4.0
2.0.4对应rocketmq 4.5.1
2.1.0对应rocketmq 4.6.0
```

#5.测试发送消息
application.yml
```
rocketmq:
  name-server: IP:9876
  producer:
    group: my-test-group
```
在springboot的测试类中
```
   @Autowired
    RocketMQTemplate rocketMQTemplate;
    @Test
    void contextLoads() {
        rocketMQTemplate.convertAndSend("test","this is content");
    }
```
#6.测试消息接收
MyConsumer消费者
```
@Component
@RocketMQMessageListener(topic = "test",consumerGroup = "my-test-group")
public class MyConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String s) {
        System.out.println("msg:  " + s);
    }
}

```

#7.查看控制台
访问IP:8080，查看控制面板

附录：
1.broker.conf
```
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

# 所属集群名字
brokerClusterName=DefaultCluster

# broker 名字，注意此处不同的配置文件填写的不一样，如果在 broker-a.properties 使用: broker-a,
# 在 broker-b.properties 使用: broker-b
brokerName=broker-a

# 0 表示 Master，> 0 表示 Slave
brokerId=0

# nameServer地址，分号分割
# namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876

# 启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句 producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
brokerIP1=宿主机IP

# 在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

# 是否允许 Broker 自动创建 Topic，建议线下开启，线上关闭 ！！！这里仔细看是 false，false，false
autoCreateTopicEnable=true

# 是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# Broker 对外服务的监听端口
listenPort=10911

# 删除文件时间点，默认凌晨4点
deleteWhen=04

# 文件保留时间，默认48小时
fileReservedTime=120

# commitLog 每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

# ConsumeQueue 每个文件默认存 30W 条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

# destroyMapedFileIntervalForcibly=120000
# redeleteHangedFileInterval=120000
# 检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
# 存储路径
# storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
# commitLog 存储路径
# storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
# 消费队列存储
# storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
# 消息索引存储路径
# storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
# checkpoint 文件存储路径
# storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
# abort 文件存储路径
# abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
# 限制的消息大小
maxMessageSize=65536

# flushCommitLogLeastPages=4
# flushConsumeQueueLeastPages=2
# flushCommitLogThoroughInterval=10000
# flushConsumeQueueThoroughInterval=60000

# Broker 的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole=ASYNC_MASTER

# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

# 发消息线程池数量
# sendMessageThreadPoolNums=128
# 拉消息线程池数量
# pullMessageThreadPoolNums=128
```


2.docker-compose.yml方式部署
```
version: '3.5'
services:
  rmqnamesrv:
    image: foxiswho/rocketmq:server
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./data/logs:/opt/logs
      - ./data/store:/opt/store
    networks:
        rmq:
          aliases:
            - rmqnamesrv

  rmqbroker:
    image: foxiswho/rocketmq:broker
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./data/logs:/opt/logs
      - ./data/store:/opt/store
      - ./data/brokerconf/broker.conf:/etc/rocketmq/broker.conf
    environment:
        NAMESRV_ADDR: "rmqnamesrv:9876"
        JAVA_OPTS: " -Duser.home=/opt"
        JAVA_OPT_EXT: "-server -Xms128m -Xmx128m -Xmn128m"
    command: mqbroker -c /etc/rocketmq/broker.conf
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqbroker

  rmqconsole:
    image: styletang/rocketmq-console-ng
    container_name: rmqconsole
    ports:
      - 8080:8080
    environment:
        JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rmqnamesrv
    networks:
      rmq:
        aliases:
          - rmqconsole

networks:
  rmq:
    name: rmq
    driver: bridge
```


3.测试消息回滚
如何发送事务消息(即半消息支持分布式事务)？ 在客户端，首先用户需要实现RocketMQLocalTransactionListener接口，并在接口类上注解声明@RocketMQTransactionListener，实现确认和回查方法；然后再使用资源模板RocketMQTemplate， 调用方法sendMessageInTransaction()来进行消息的发布。 注意：这个方法通过指定发送者组名与具体的声明了txProducerGroup的TransactionListener进行关联，您也可以不指定这个值，从而使用默认的事务发送者组。

3.1 测试生产消息类
```
    @Autowired
    RocketMQTemplate rocketMQTemplate;
    @Test
    void contextLoads() {
        Message<String> sendMsg = MessageBuilder.withPayload("send msg").build();
        rocketMQTemplate.sendMessageInTransaction("tx-test-group","test1",sendMsg,null);
        System.out.println("发送完成");
        try {
            //进行阻塞。防止方法执行完 无法看到消息监听
            Thread.sleep(5000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

```
3.2 测试事务监听类
```
import org.apache.rocketmq.spring.annotation.RocketMQTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionState;
import org.apache.rocketmq.spring.support.RocketMQHeaders;
import org.springframework.messaging.Message;
import java.util.HashMap;
import java.util.Map;

@RocketMQTransactionListener(txProducerGroup = "tx-test-group")
public class RocketTransactionListener implements RocketMQLocalTransactionListener {


    private static Map<String, RocketMQLocalTransactionState> STATE_MAP = new HashMap<>();

    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message message, Object o) {
        String transId = (String)message.getHeaders().get(RocketMQHeaders.TRANSACTION_ID);

        try {
            System.out.println("执行操作");
            Thread.sleep(500);
//            测试回滚打开下方注释
//            int a = 1/0;
            // 设置事务状态
            STATE_MAP.put(transId, RocketMQLocalTransactionState.COMMIT);
            // 返回事务状态给生产者， 若测试回查改成返回RocketMQLocalTransactionState.UNKNOWN
            return RocketMQLocalTransactionState.COMMIT;
        } catch (Exception e) {
            e.printStackTrace();
        }

        STATE_MAP.put(transId, RocketMQLocalTransactionState.ROLLBACK);
        return RocketMQLocalTransactionState.ROLLBACK;
    }

    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message message) {
        String transId = (String)message.getHeaders().get(RocketMQHeaders.TRANSACTION_ID);
        System.out.println("回查消息 -> transId = " + transId + ", state = " + STATE_MAP.get(transId));
        return STATE_MAP.get(transId);
    }
}

```
3.3 测试消息消费类
```
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;

@Component
@RocketMQMessageListener(topic = "test1",consumerGroup = "tx-test-group")
public class MyConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String s) {
        System.out.println("msg:  "+s);
    }
}

```

4.测试消息延迟
RocketMQ不支持任意时间的延时，只支持以下几个固定的延时等级
private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
4.1 发送消息
```
        Message<String> sendMsg = MessageBuilder.withPayload("delay msg").build();
        rocketMQTemplate.syncSend("delay-test-group",sendMsg,2000,5);
        System.out.println(DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss").format(LocalDateTime.now()));
        System.out.println("发送完成");
```
注：syncSend 第四个参数设置消息延迟等级，根据messageDelayLevel的顺序
4.2 消费者
```
import org.apache.rocketmq.spring.annotation.ConsumeMode;
import org.apache.rocketmq.spring.annotation.MessageModel;
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Component
@RocketMQMessageListener(
        topic = "delay-test-group",
        consumerGroup = "delay-test-group",
        selectorExpression = "*",
        consumeMode = ConsumeMode.ORDERLY,
        messageModel = MessageModel.CLUSTERING,
        consumeThreadMax = 1
)
public class DelayRocketConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String msg) {
        System.out.println("接收到消息 -> " + msg);
        System.out.println(DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss").format(LocalDateTime.now()));
    }
}
```

RocketMQ的延迟等级可以进行修改，以满足自己的业务需求，可以修改/添加新的level。例如：你想支持1天的延迟，修改最后一个level的值为1d，这个时候依然是18个level；也可以增加一个1d，这个时候总共就有19个level。

打开RocketMQ的配置文件，修改 messageDelayLevel 属性
```
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
storePathRootDir = /app/rocketmq/data
messageDelayLevel=90s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```
这次将延时等级1修改成了90s，生产者发送消息后需要90s后再进行消息投递。修改完成后重启RocketMQ。
```
nohup sh mqbroker -n localhost:9876 -c ../conf/broker.conf &
```
