# Rocketmq源码构建

参考地址：http://www.mstacks.com/137/1467.html#content1467

# 1.下载源码 构建环境
```
git clone https://github.com/apache/rocketmq
```
# 2.进到rocketmq目录 执行打包
```
mvn clean compile -Dmaven.test.skip=true package
```
RocketMQ源码的模块结构如下：
```
	broker：存放RocketMQ的Broker相关的代码，这里的代码可以用来启动Broker进程；
	client：存放RocketMQ的Producer、Consumer这些客户端的代码，生产消息、消费消息的代码都在里面；
	common：存放公共代码；
	dev：存放开发相关的一些信息；
	distribution：存放用来部署RocketMQ的一些东西，比如bin目录 、conf目录等等；
	example：存放RocketMQ的一些例子；
	filter：存放RocketMQ的与过滤器相关的代码；
	logappender和logging：存放RocketMQ的日志打印相关的东西；
	namesvr：存放NameServer的源码；
	openmessaging：这是开放消息标准，先忽略；
	remoting：存放RocketMQ的远程网络通信模块的代码，基于netty实现；
	srvutil：存放一些工具类；
	store：存放Broker上存储相关的一些源码；
	style、test、tools：存放checkstyle代码检查的东西，一些测试相关的类，还有就是tools里放的一些命令行监控工具类。
```

# 3.建立一个ROCKETMQ_HOME目录，加入配置文件
在ROCKETMQ_HOME根目录下创建conf、logs、store三个文件夹，因为后续NameServer运行是需要使用这些目录的。然后我们把RocketMQ源码目录中的distrbution模块下的broker.conf、logback_namesvr.xml两个配置文件拷贝到上述的conf目录中去，接着就需要修改这两个配置文件的内容
logback_namesvr.xml：修改里面的所有${user.home}，替换为你的ROCKETMQ_HOME根目录。

broker.conf：修改为以下内容
```
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
# 添加此项，broker连接到本地的namesrv上
namesrvAddr = 127.0.0.1:9876
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH

###########################################添加以下配置，路径请修改为自己的正确路径
# 配置存储位置
storePathRootDir = D:/workplace/rocketmq/rocket_home/store
# commitlog 存储路径
storePathCommitLog = D:/workplace/rocketmq/rocket_home/store/commitlog
# 消费队列存储路径
storePathConsumeQueue = D:/workplace/rocketmq/rocket_home/store/consumequeue
# 消息索引存储路径
storePathIndex = D:/workplace/rocketmq/rocket_home/store/index
# checkpoint文件存储路径
storeCheckPoint = D:/workplace/rocketmq/rocket_home/store/checkpoint
# abort文件存储路径
abortFile = D:/workplace/rocketmq/rocket_home/store/abort

```
# 4.启动NameServer
注意启动时，要先配置好ROCKETMQ_HOME环境变量，可以在IDE中进行配置：
添加Application，
```
Main class：org.apache.rocketmq.namesrv.NamesrvStartup
Working Direcory: D:\workplace\rocketmq
Environment variables: ROCKETMQ_HOME=D:\workplace\rocketmq\rocket_home
Use classpath moudle: rocketmq-namesrv
```
启动成功之后，在Intellij IDEA的命令行里就会看到下面的提示：
```
Connected to the target VM, address: '127.0.0.1:54473', transport: 'socket'
The Name Server boot success. serializeType=JSON
```

# 5.启动broker
添加Application，
```
Main class：org.apache.rocketmq.broker.BrokerStartup
R破果然没arguments： -c D:\workplace\rocketmq\rocket_home/conf/broker.conf
Working Direcory: D:\workplace\rocketmq
Environment variables: ROCKETMQ_HOME=D:\workplace\rocketmq\rocket_home
Use classpath moudle: rocketmq-broker
```
启动成功之后，在Intellij IDEA的命令行里就会看到下面的提示：
```
Connected to the target VM, address: '127.0.0.1:54473', transport: 'socket'
The Name Server boot success. serializeType=JSON
```
# 6.修改Producer(example模块下org.apache.rocketmq.example.quickstart)
```
package org.apache.rocketmq.example.quickstart;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

/**
 * This class demonstrates how to send messages to brokers using provided {@link DefaultMQProducer}.
 */
public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {

        /*
         * Instantiate with a producer group name.
         */
        DefaultMQProducer producer = new DefaultMQProducer("code_group_test");

        /*
         * Specify name server addresses.
         * <p/>
         *
         * Alternatively, you may specify name server addresses via exporting environmental variable: NAMESRV_ADDR
         * <pre>
         * {@code
         * producer.setNamesrvAddr("name-server1-ip:9876;name-server2-ip:9876");
         * }
         * </pre>
         */
        producer.setNamesrvAddr("127.0.0.1:9876");
        /*
         * Launch the instance.
         */
        producer.start();

        for (int i = 0; i < 1000; i++) {
            try {

                /*
                 * Create a message instance, specifying topic, tag and message body.
                 */
                Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
                );

                /*
                 * Call send message to deliver message to one of brokers.
                 */
                SendResult sendResult = producer.send(msg);

                System.out.printf("send msg: %s%n", sendResult);
            } catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }

        /*
         * Shut down once the producer instance is not longer in use.
         */
        producer.shutdown();
    }
}

```

# 7.修改Consumer(example模块下org.apache.rocketmq.example.quickstart)
```
package org.apache.rocketmq.example.quickstart;

import java.util.List;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

/**
 * This example shows how to subscribe and consume messages using providing {@link DefaultMQPushConsumer}.
 */
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {

        /*
         * Instantiate with specified consumer group name.
         */
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("code_group_test");

        /*
         * Specify name server addresses.
         * <p/>
         *
         * Alternatively, you may specify name server addresses via exporting environmental variable: NAMESRV_ADDR
         * <pre>
         * {@code
         * consumer.setNamesrvAddr("name-server1-ip:9876;name-server2-ip:9876");
         * }
         * </pre>
         */
        consumer.setNamesrvAddr("127.0.0.1:9876");
        /*
         * Specify where to start in case the specified consumer group is a brand new one.
         */
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        /*
         * Subscribe one more more topics to consume.
         */
        consumer.subscribe("TopicTest", "*");

        /*
         *  Register callback to execute on arrival of messages fetched from brokers.
         */
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        /*
         *  Launch the consumer instance.
         */
        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
}

```