
# 搭建Mysql

准备/home/mysql/conf/my.cnf
开启bin日志
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
binlog-format=ROW
server_id=5
```

```
docker run -p 3306:3306 --name mysql -v /home/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

注： server_id与canal的server_id请勿冲突
使用命令show variables like 'log_%';进行查看，为ON表明binlog开启


创建canal账号
```
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```
将配置文件拷入容器中,并重启
```
docker cp /home/mysql/conf/my.cnf mysql:/etc/mysql/my.cnf
docker restart mysql
```


# 搭建canal
将数据库连接到canal
```
docker run --name mycanal -e canal.auto.scan=false  -e canal.destinations=test  -e canal.instance.master.address=db:3306   -e canal.instance.dbUsername=canal   -e canal.instance.dbPassword=canal    -e canal.instance.connectionCharset=UTF-8  -e canal.instance.tsdb.enable=true  -e canal.instance.gtidon=false    -e canal.admin.port=11110 -e canal.admin.user=admin -e canal.admin.passwd=4ACFE3202A5FF5CF467898FC58AAB1D615029441  -p 11112:11112  -p 11111:11111 -p 11110:11110 --link mysql:db   -d canal/canal-server:v1.1.4
```

# maven项目引入jar
```
   <dependency>
            <groupId>com.alibaba.otter</groupId>
            <artifactId>canal.client</artifactId>
            <version>1.1.4</version>
   </dependency>
```

# 编写监听测试代码SimpleCanalClientExample
```
import java.net.InetSocketAddress;
import java.util.List;
import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.protocol.CanalEntry;
import com.alibaba.otter.canal.protocol.Message;
import com.alibaba.otter.canal.protocol.CanalEntry.Entry;
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;
import com.alibaba.otter.canal.protocol.CanalEntry.EventType;
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;
import com.alibaba.otter.canal.protocol.CanalEntry.RowData;

public class SimpleCanalClientExample {
    public static void main(String args[]) {
        CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress("服务器IP",
                11111), "test", "canal", "canal");
        int batchSize = 1000;
        int emptyCount = 0;
        try {
            connector.connect();
            connector.subscribe(".*\\..*");
            connector.rollback();
            int totalEmptyCount = 220;
            while (emptyCount < totalEmptyCount) {
                Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
                long batchId = message.getId();
                int size = message.getEntries().size();
                if (batchId == -1 || size == 0) {
                    emptyCount++;
                    System.out.println("empty count : " + emptyCount);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                    }
                } else {
                    emptyCount = 0;
                    // System.out.printf("message[batchId=%s,size=%s] \n", batchId, size);
                    printEntry(message.getEntries());
                }

                connector.ack(batchId); // 提交确认
                // connector.rollback(batchId); // 处理失败, 回滚数据
            }

            System.out.println("empty too many times, exit");
        } finally {
            connector.disconnect();
        }
    }

    private static void printEntry(List<Entry> entrys) {
        for (Entry entry : entrys) {
            if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                continue;
            }

            RowChange rowChage = null;
            try {
                rowChage = RowChange.parseFrom(entry.getStoreValue());
            } catch (Exception e) {
                throw new RuntimeException("ERROR ## parser of eromanga-event has an error , data:" + entry.toString(),
                        e);
            }

            EventType eventType = rowChage.getEventType();
            System.out.println(String.format("================&gt; binlog[%s:%s] , name[%s,%s] , eventType : %s",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),
                    eventType));

            for (RowData rowData : rowChage.getRowDatasList()) {
                if (eventType == EventType.DELETE) {
                    printColumn(rowData.getBeforeColumnsList());
                } else if (eventType == EventType.INSERT) {
                    printColumn(rowData.getAfterColumnsList());
                } else {
                    System.out.println("-------&gt; before");
                    printColumn(rowData.getBeforeColumnsList());
                    System.out.println("-------&gt; after");
                    printColumn(rowData.getAfterColumnsList());
                }
            }
        }
    }

    private static void printColumn(List<CanalEntry.Column> columns) {
        for (CanalEntry.Column column : columns) {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
        }
    }

}

```


附录：
canal.properties配置说明：
```
#################################################
#########       common argument     #############
#################################################
# tcp bind ip
canal.ip =      //canal server绑定的本地IP信息，如果不配置，默认选择一个本机IP进行启动服务
# register ip to zookeeper
canal.register.ip =    //canal server注册到外部zookeeper、admin的ip信息 (针对docker的外部可见ip)
canal.port = 11111     //canal server提供socket服务的端口
canal.metrics.pull.port = 11112
# canal instance user/passwd
#canal.user = canal    //canal数据端口订阅的ACL配置 (v1.1.4新增)如果为空，代表不开启
#canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458 //canal数据端口订阅的ACL配置 (v1.1.4新增)如果为空，代表不开启

# canal admin config
canal.admin.manager = 127.0.0.1:8089   //canal链接canal-admin的地址 (v1.1.4新增)
canal.admin.port = 11110              //admin管理指令链接端口 (v1.1.4新增)
canal.admin.user = admin               //admin管理指令链接的ACL配置 (v1.1.4新增)
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441 //admin管理指令链接的ACL配置 (v1.1.4新增)

canal.zkServers =       //canal server链接zookeeper集群的链接信息
# flush data to zk
canal.zookeeper.flush.period = 1000   //canal持久化数据到zookeeper上的更新频率，单位毫秒
canal.withoutNetty = false
# tcp, kafka, RocketMQ
canal.serverMode = tcp
# flush meta cursor/parse position to file
canal.file.data.dir = ${canal.conf.dir}//主要针对h2-tsdb.xml时对应h2文件的存放目录,默认为conf/xx/h2.mv.db
canal.file.flush.period = 1000
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 16384 //canal内存store中可缓存buffer记录数，需要为2的指数
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024 //内存记录的单位大小，默认1KB，和buffer.size组合决定最终的内存使用大小
## meory store gets mode used MEMSIZE or ITEMSIZE
canal.instance.memory.batch.mode = MEMSIZE //canal内存store中数据缓存模式
//1. ITEMSIZE : 根据buffer.size进行限制，只限制记录的数量
//2. MEMSIZE : 根据buffer.size  * buffer.memunit的大小，限制缓存记录的大小
canal.instance.memory.rawEntry = true

## detecing config
canal.instance.detecting.enable = false  //是否开启心跳检查
#canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate key update x=now()  //  心跳检查sql
canal.instance.detecting.sql = select 1 
canal.instance.detecting.interval.time = 3  //心跳检查频率，单位秒
canal.instance.detecting.retry.threshold = 3  //心跳检查失败重试次数
canal.instance.detecting.heartbeatHaEnable = false //心跳检查失败后，是否开启自动mysql自动切换
说明：比如心跳检查失败超过阀值后，如果该配置为true，canal就会自动链到mysql备库获取binlog数据

# support maximum transaction size, more than the size of the transaction will be cut into multiple transactions delivery
canal.instance.transaction.size =  1024
# mysql fallback connected to new master should fallback times
canal.instance.fallbackIntervalInSeconds = 60 //canal发生mysql切换时，在新的mysql库上查找binlog时需要往前查找的时间，单位秒
说明：mysql主备库可能存在解析延迟或者时钟不统一，需要回退一段时间，保证数据不丢

# network config
canal.instance.network.receiveBufferSize = 16384 //网络链接参数，SocketOptions.SO_RCVBUF
canal.instance.network.sendBufferSize = 16384 //网络链接参数，SocketOptions.SO_SNDBUF
canal.instance.network.soTimeout = 30 //网络链接参数，SocketOptions.SO_TIMEOUT

# binlog filter config
canal.instance.filter.druid.ddl = true //是否使用druid处理所有的ddl解析来获取库和表名
canal.instance.filter.query.dcl = false //是否忽略dcl语句
canal.instance.filter.query.dml = false //是否忽略dml语句
(mysql5.6之后，在row模式下每条DML语句也会记录SQL到binlog中,可参考MySQL文档)
canal.instance.filter.query.ddl = false //是否忽略ddl语句
canal.instance.filter.table.error = false //是否忽略binlog表结构获取失败的异常
(主要解决回溯binlog时,对应表已被删除或者表结构和binlog不一致的情况)
canal.instance.filter.rows = false //是否dml的数据变更事件(主要针对用户只订阅ddl/dcl的操作)
canal.instance.filter.transaction.entry = false //是否忽略事务头和尾,比如针对写入kakfa的消息时，不需要写入TransactionBegin/Transactionend事件

# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED //支持的binlog format格式列表(otter会有支持format格式限制)
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB //支持的binlog image格式列表(otter会有支持format格式限制)

# binlog ddl isolation
canal.instance.get.ddl.isolation = false //ddl语句是否单独一个batch返回
(比如下游dml/ddl如果做batch内无序并发处理,会导致结构不一致)

# parallel parser config
canal.instance.parser.parallel = true //是否开启binlog并行解析模式
## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
#canal.instance.parser.parallelThreadSize = 16
## disruptor ringbuffer size, must be power of 2
canal.instance.parser.parallelBufferSize = 256 //binlog并行解析的异步ringbuffer队列(必须为2的指数)

# table meta tsdb info
canal.instance.tsdb.enable = true //是否开启tablemeta的tsdb能力
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360

#################################################
#########       destinations        #############
#################################################
canal.destinations =  //当前server上部署的instance列表
# conf root dir
canal.conf.dir = ../conf    //conf/目录所在的路径
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true //开启instance自动扫描
如果配置为true，canal.conf.dir目录下的instance配置变化会自动触发
canal.auto.scan.interval = 5  //instance自动扫描的间隔时间，单位秒

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml //v1.0.25版本新增,全局的tsdb配置方式的组件文件
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml

canal.instance.global.mode = manager //全局配置加载方式
canal.instance.global.lazy = false //全局lazy模式
canal.instance.global.manager.address = ${canal.admin.manager} //全局的manager配置方式的链接信息
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
canal.instance.global.spring.xml = classpath:spring/file-instance.xml //全局的spring配置方式的组件文件
#canal.instance.global.spring.xml = classpath:spring/default-instance.xml

##################################################
#########                    MQ                      #############
##################################################
canal.mq.servers = 127.0.0.1:6667 //kafka/rocketmq 集群配置: 192.168.1.117:9092,192.168.1.118:9092
canal.mq.retries = 0  //发送失败重试次数
canal.mq.batchSize = 16384  //kafka为ProducerConfig.BATCH_SIZE_CONFIG
canal.mq.maxRequestSize = 1048576 //kafka为ProducerConfig.MAX_REQUEST_SIZE_CONFIG
canal.mq.lingerMs = 100 //kafka为ProducerConfig.LINGER_MS_CONFIG , 如果是flatMessage格式建议将该值调大, 如: 200
canal.mq.bufferMemory = 33554432 //kafka为ProducerConfig.BUFFER_MEMORY_CONFIG
canal.mq.canalBatchSize = 50 //获取canal数据的批次大小
canal.mq.canalGetTimeout = 100 //获取canal数据的超时时间
canal.mq.flatMessage = true //是否为json格式
canal.mq.compressionType = none
canal.mq.acks = all  //kafka为ProducerConfig.ACKS_CONFIG
#canal.mq.properties. =
canal.mq.producerGroup = test //kafka无意义
```