# docker简易搭建Mycat

# 1.配置文件准备
1.1 schema.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
  <!-- 注意TESTDB是MyCat设置的抽象数据库名，对应我们配置的多个真实数据库 -->
  <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
    <!-- user表对应dn1节点 -->
    <table name="user" dataNode="dn1" />
    <!-- blog表对应dn1/dn2/dn3节点，rule-blog表示对blog表的路由规则名称 -->
    <table name="blog" dataNode="dn1,dn2,dn3" rule="rule-blog" />
  </schema>

  <!-- 为实例下面的各个库设置节点 -->
  <dataNode name="dn1" dataHost="localhost1" database="db1" />
  <dataNode name="dn2" dataHost="localhost1" database="db2" />
  <dataNode name="dn3" dataHost="localhost1" database="db3" />

  <!-- 配置真实数据库实例信息 -->
  <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
    <heartbeat>select user()</heartbeat>
    <!-- 配置数据库url、用户名、密码 -->
    <writeHost host="server1" url="IP:3307" user="root" password="123456" />
  </dataHost>
</mycat:schema>
```



1.2 server.xml
通过修改conf/server.xml配置MyCat对外服务信息，主要就是用户名、密码、以及上面指定的抽象数据库名称TESTDB。
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
  <!-- system部分采用默认即可 -->
  <system>
    <property name="nonePasswordLogin">0</property>    <!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
    <property name="useHandshakeV10">1</property>
    <property name="useSqlStat">0</property>    <!-- 1为开启实时统计、0为关闭 -->
    <property name="useGlobleTableCheck">0</property>    <!-- 1为开启全加班一致性检测、0为关闭 -->
    <property name="sequnceHandlerType">2</property>     <!-- 主键方式 -->
    <property name="subqueryRelationshipCheck">false</property>    <!-- 子查询中存在关联查询的情况下,检查关联字段中是否有分片字段 .默认 false -->
    <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena | type 2 NettyBufferPool -->
    <property name="processorBufferPoolType">0</property>
    <!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
    <property name="handleDistributedTransactions">0</property>
    <!--off heap for merge/order/group/limit      1开启   0关闭-->
    <property name="useOffHeapForMerge">1</property>
    <!--单位为m-->
    <property name="memoryPageSize">64k</property>
    <!--单位为k-->
    <property name="spillsFileBufferSize">1k</property>
    <property name="useStreamOutput">0</property>
    <!--单位为m-->
    <property name="systemReserveMemorySize">384m</property>
    <!--是否采用zookeeper协调切换  -->
    <property name="useZKSwitch">false</property>
  </system>

  <!-- 设置访问的用户名密码 -->
  <user name="root">
    <property name="password">123456</property>
    <!-- 注意此处是之前设定的抽象数据库名称 -->
    <property name="schemas">TESTDB</property>
  </user>
</mycat:server>
```

Balance均衡策略设置：
- 1) balance=0  不开启读写分离机制，所有读操作都发送到当前可用的writehost；
- 2) balance=1  全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且M1与 M2互为主备)，正常情况下，M2,S1,S2都参与select语句的负载均衡。
- 3) balance=2  所有读操作都随机的在readhost和writehost上分发；
- 4) balance=3  所有读请求随机的分发到wiriterHost对应的readhost执行，writerHost不负担读压力。
writeType 写入策略设置
- 1) writeType=0，所有写操作发送到配置的第一个writeHost；
- 2) writeType=1，所有写操作都随机的发送到配置的writeHost；
- 3) writeType=2，不执行写操作。
switchType 策略设置
- 1) switchType=-1，表示不自动切换；
- 2) switchType=1，默认值，自动切换；
- 3) switchType=2，基于MySQL 主从同步的状态决定是否切换；
- 4) switchType=3，基于MySQL galary cluster的切换机制（适合集群）（1.4.1），心跳语句为 show status like 'wsrep%'。


1.3 rule.xml
在schema.xml中我们已经制定了blog表存储的节点，且设置了路由规则的名称rule-blog，然后我们设置该规则具体的策略。
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">
<mycat:rule xmlns:mycat="http://io.mycat/">
  <!-- 指定要设置的规则 -->
  <tableRule name="rule-blog">
    <rule>
      <!-- 规则生效的列 -->
      <columns>id</columns>
      <!-- 对应的路由算法 -->
      <algorithm>rule-blog-algorithm</algorithm>
    </rule>
  </tableRule>

  <!-- 配置算法具体实现方式 -->
  <function name="rule-blog-algorithm" class="io.mycat.route.function.PartitionByMod">
    <property name="count">3</property>    <!-- 表示对id进行除3取模分表 -->
  </function>

</mycat:rule>
```

1.4 wrapper.conf
修改java启动所需要的内存，以防内存不够导致无法启动
```
#********************************************************************
# Wrapper Properties
#********************************************************************
# Java Application
wrapper.java.command=java
wrapper.working.dir=..

# Java Main class.  This class must implement the WrapperListener interface
#  or guarantee that the WrapperManager class is initialized.  Helper
#  classes are provided to do this for you.  See the Integration section
#  of the documentation for details.
wrapper.java.mainclass=org.tanukisoftware.wrapper.WrapperSimpleApp
set.default.REPO_DIR=lib
set.APP_BASE=.

# Java Classpath (include wrapper.jar)  Add class path elements as
#  needed starting from 1
wrapper.java.classpath.1=lib/wrapper.jar
wrapper.java.classpath.2=conf
wrapper.java.classpath.3=%REPO_DIR%/*

# Java Library Path (location of Wrapper.DLL or libwrapper.so)
wrapper.java.library.path.1=lib

# Java Additional Parameters
#wrapper.java.additional.1=
wrapper.java.additional.1=-DMYCAT_HOME=.
wrapper.java.additional.2=-server
wrapper.java.additional.3=-XX:+AggressiveOpts
wrapper.java.additional.4=-XX:MaxDirectMemorySize=2G
wrapper.java.additional.5=-Dcom.sun.management.jmxremote
wrapper.java.additional.6=-Dcom.sun.management.jmxremote.port=1984
wrapper.java.additional.7=-Dcom.sun.management.jmxremote.authenticate=false
wrapper.java.additional.8=-Dcom.sun.management.jmxremote.ssl=false
wrapper.java.additional.9=-Xmx256M #修改启动内存 防止过小导致无法启动
wrapper.java.additional.10=-Xms128M #修改启动内存 防止过小导致无法启动

# Initial Java Heap Size (in MB)
#wrapper.java.initmemory=3

# Maximum Java Heap Size (in MB)
#wrapper.java.maxmemory=64

# Application parameters.  Add parameters as needed starting from 1
wrapper.app.parameter.1=io.mycat.MycatStartup
wrapper.app.parameter.2=start

#********************************************************************
# Wrapper Logging Properties
#********************************************************************
# Format of output for the console.  (See docs for formats)
wrapper.console.format=PM

# Log Level for console output.  (See docs for log levels)
wrapper.console.loglevel=INFO

# Log file to use for wrapper output logging.
wrapper.logfile=logs/wrapper.log

# Format of output for the log file.  (See docs for formats)
wrapper.logfile.format=LPTM

# Log Level for log file output.  (See docs for log levels)
wrapper.logfile.loglevel=INFO

# Maximum size that the log file will be allowed to grow to before
#  the log is rolled. Size is specified in bytes.  The default value
#  of 0, disables log rolling.  May abbreviate with the 'k' (kb) or
#  'm' (mb) suffix.  For example: 10m = 10 megabytes.
wrapper.logfile.maxsize=512m

# Maximum number of rolled log files which will be allowed before old
#  files are deleted.  The default value of 0 implies no limit.
wrapper.logfile.maxfiles=30

# Log Level for sys/event log output.  (See docs for log levels)
wrapper.syslog.loglevel=NONE

#********************************************************************
# Wrapper Windows Properties
#********************************************************************
# Title to use when running as a console
wrapper.console.title=Mycat-server

#********************************************************************
# Wrapper Windows NT/2000/XP Service Properties
#********************************************************************
# WARNING - Do not modify any of these properties when an application
#  using this configuration file has been installed as a service.
#  Please uninstall the service before modifying this section.  The
#  service can then be reinstalled.

# Name of the service
wrapper.ntservice.name=mycat

# Display name of the service
wrapper.ntservice.displayname=Mycat-server

# Description of the service
wrapper.ntservice.description=The project of Mycat-server

# Service dependencies.  Add dependencies as needed starting from 1
wrapper.ntservice.dependency.1=

# Mode in which the service is installed.  AUTO_START or DEMAND_START
wrapper.ntservice.starttype=AUTO_START

# Allow the service to interact with the desktop.
wrapper.ntservice.interactive=false

wrapper.ping.timeout=120
configuration.directory.in.classpath.first=conf

```

# 2.启动Mysql容器
2.1 准备Mysql主从复制环境
2.1.1 my.cnf
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
binlog-format=ROW
server_id=5
max_allowed_packet=512M
```
2.1.2 mysalve.cnf
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
binlog-format=ROW
server_id=6
max_allowed_packet=512M
```

2.2 mysql容器启动
mysql
```
docker run -p 3307:3306 --name mysql -v /home/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
mysql-salve
```
docker run -p 3308:3306 --name mysql-salve -v /home/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

2.3 配置重启容器
将配置文件拷入容器中,并重启
```
docker cp /home/mysql/conf/my.cnf mysql:/etc/mysql/my.cnf
docker cp /home/mysql/conf/mysalve.cnf mysql-salve:/etc/mysql/my.cnf
docker restart mysql
docker restart mysql-salve
```

# 3.启动Mycat
```
 docker run -d -p 8066:8066 -p 9066:9066 --name mycat  -v /home/software/mycat/rule.xml:/usr/local/mycat/conf/rule.xml -v /home/software/mycat/server.xml:/usr/local/mycat/conf/server.xml -v /home/software/mycat/schema.xml:/usr/local/mycat/conf/schema.xml -v /home/software/mycat/wrapper.conf:/usr/local/mycat/conf/wrapper.conf longhronshens/mycat-docker
``` 

注：9066使用：mysql -h127.0.0.1 -utest -ptest -P9066，8066用于管理端口操作

MyCAT配置目录详解如下：
```
bin程序目录，存放了 window版本和linux版本启动脚本，除了提供封装服务的版本之外，也提供了 nowrap的 shell脚本命令，方便大家选择和修改，进入到bin目录：
Linux 下运行：./mycat console,首先要 chmod +x *
mycat 支持的命令{ console | start | stop | restart | status | dump }
conf目录下存放配置文件，其中：
	server.xm  丨Mycat服务器参数调整和用户授权的配置文件；
	schema.xm丨逻辑库定义和表及分片定义的配置文件；
	rule.xml    |分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下，配置文件修改，需要重启Mycat或者通过9066端口 reload；
	lib目录下主要存放mycat依赖的一些jar文件；
	日志存放在logs/mycat.log中，每天一个文件，日志的配置是在conf/log4j.xml中，根据自己的需要，可以调整输出级别为debug , debug级别下；
	Catlet  |支持跨分片复杂SQL实现以及存储过程支持。
```





# 4.数据库准备
规划设计三个库db1、db2、db3
使用Navicat工具连接3307端口数据库创建db1、db2、db3
有两个表用户表user和博客表blog，用户表存储于db1，博客表存储于db1/db2/db3。

在db1执行如下语句创建user表
```
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增序号',
  `name` varchar(255) DEFAULT '' COMMENT '姓名',
  `password` varchar(255) DEFAULT '' COMMENT '密码',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

然后再db1/db2/db3执行如下语句创建blog表
```
CREATE TABLE `blog` (
  `id` int(11) NOT NULL COMMENT '序号',
  `title` varchar(255) DEFAULT '' COMMENT '博客标题',
  `content` varchar(255) DEFAULT '' COMMENT '博客内容',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

# 5.连接Mycat
使用navicat 通过上面配置的MyCat用户名密码连接MyCat
```
用户名 root，密码 123456 端口 8066
```
连接成功后，会发现一个数据库TESTDB，及两张表user/blog，TESTDB即为MyCat抽象出来的数据库，操作TESTDB中的数据会按照当时设定的规则，实际操作会发生在db1/db2/db3中，这个我们不必关心，MyCat会自动为我们实现。
 
# 6.数据插入与查询测试
我们在TESTDB执行6次insert语句。
```
insert into blog(id,title,content)values(1,"title1","content1")
insert into blog(id,title,content)values(2,"title2","content2")
insert into blog(id,title,content)values(3,"title3","content3")
insert into blog(id,title,content)values(4,"title4","content4")
insert into blog(id,title,content)values(5,"title5","content5")
insert into blog(id,title,content)values(6,"title6","content6")

insert into user(id,name,password)values(1,"u1","123456");
```
在TESTDB查询有6条数据，说明插入成功。
而db1有id为3/6的数据。
db2有id为1/4的数据。
db3有id为2/5的数据。
说明MyCat自动为我们取模了

```
select * from blog 可以查出6条数据
select * from user 可以查出1条数据
```



附录：
https://www.cnblogs.com/wxzhe/p/10265070.html
server.xml可以配置 IP白名单 SQL黑名单

# 数据库中间件区别：
<table>
	<thead><tr><th>&nbsp;</th><th><em>Sharding-JDBC</em></th><th><em>mycat</em></th><th><em>drds</em></th></tr></thead>
	<tbody><tr><td>性能</td><td>高</td><td>中</td><td>高</td></tr><tr><td>应用场景限制</td><td>java应用</td><td>无</td><td>无</td></tr>
		   <tr><td>是否支持自定义sharding路由</td><td>是</td><td>是</td><td>是</td></tr>
		   <tr><td>最大支持sharding路由维度</td><td>2</td><td>1</td><td>2</td></tr>
		   <tr><td>分布式事务</td><td>开发中</td><td>支持弱xa、支持XA分布式事务（1.6.5）</td><td>支持以下分布式事务策略：FREE、2PC、XA、FLEXIBLE</td></tr>
		   <tr><td>限制</td><td>不支持子语句,不支持UNION 和 UNION ALL,不支持批量插入,不支持DISTINCT聚合</td><td>详见<a href="http://www.mycat.io/document/mycat-definitive-guide.pdf">《MYCAT权威指南》</a>——5.6 Mycat 目前存在的限制</td><td>未明确说明</td></tr><tr><td>是否开源</td><td>是</td><td>是</td><td>否</td></tr>
	</tbody>
</table>

# 数据库拆分六大原则
- 1.优先考虑缓存降低对数据库的读操作。
- 2.再考虑读写分离，降低数据库写操作。
- 3.最后开始数据拆分,切分模式： 首先垂直（纵向）拆分、再次水平拆分。
- 4.首先考虑按照业务垂直拆分。
- 5.再考虑水平拆分：先分库(设置数据路由规则，把数据分配到不同的库中)
- 6.最后再考虑分表，单表拆分到数据1000万以内。


https://blog.csdn.net/qq_41143671/article/details/112960048

# 常用的分片规则
https://www.bbsmax.com/A/q4zVZLXbzK/

# 拆分分类
1.垂直分表定义：将一个表按照字段分成多表，每个表存储其中一部分字段。
它带来的提升是：
- 1.为了避免IO争抢并减少锁表的几率，查看详情的用户与商品信息浏览互不影响；
- 2.充分发挥热门数据的操作效率，商品信息的操作的高效率不会被商品描述的低效率所拖累。

```
为什么大字段IO效率低 ：
	第一是由于数据量本身大，需要更长的读取时间；
	第二是跨页，页是数据库存储单位，很多查找及定位操作都是以页为单位，单页内的数据行越多数据库整体性能越好，而大字段占用空间大，单页内存储行数少，因此IO效率较低。
	第三，数据库以行为单位将数据加载到内存中，这样表中字段长度较短且访问频率较高，内存能加载更多的数据，命中率更高，减少了磁盘IO，从而提升了数据库性能。
```
一般来说，某业务实体中的各个数据项的访问频次是不一样的，部分数据项可能是占用存储空间比较大的BLOB或是TEXT。例如上例中的商品描述。所以，当表数据量很大时，可以将表按字段切开，将热门字段、冷门字段分开放置在不同库中，这些库可以放在不同的存储设备上，避免IO争抢。垂直切分带来的性能提升主要集中在热门数据的操作效率上，而且磁盘争用情况减少 。
通常，垂直分表的原则 ：
- 查询使用频率较高的列放在一张表中；
- 把不常用的字段单独放在一张表；
- 把 text，blob 等大字段拆分出来放在附表中。

2.垂直分库
通过垂直分表性能得到了一定程度的提升，但是还没有达到要求，并且磁盘空间也快不够了，因为数据还是始终限制在一台服务器，库内垂直分表只解决了单一表数据量过大的问题，但没有将表分布到不同的服务器上，因此每个表还是竞争同一个物理机的 CPU、内存、网络IO、磁盘
垂直分库 是指按照业务类型对表进行分类，分布到不同的数据库上面，每个库可以放在不同的服务器上，它的核心理念是 专库专用 。
它带来的提升是：
- 解决业务层面的耦合，业务清晰；
- 能对不同业务的数据进行分级管理、维护、监控、扩展等；
- 高并发场景下，垂直分库一定程度的提升IO、数据库连接数、降低单机硬件资源的瓶颈。
- 垂直分库通过将表按业务分类，然后分布在不同数据库，并且可以将这些数据库部署在不同服务器上，从而达到多个服务器共同分摊压力的效果，但是依然没有解决单表数据量过大的问题。

3.水平分库
水平分库的定义：
水平分库是把同一个表的数据按一定规则分配到不同的数据库中，每个库可以放在不同的服务器上。
```
	与 垂直分库 的区别？
	垂直分库是把不同表拆分到不同数据库中，而水平分库是对数据行的拆分，不涉及表结构。
```
水平分库的优点：
解决了单库大数据，高并发的性能瓶颈。
提高了系统的稳定性及可用性。
`稳定性体现在IO冲突减少，锁定减少，可用性指某个库出问题，部分可用。`

水平分库的缺点：
当一个应用难以再细粒度的垂直切分，或切分后数据量行数巨大，存在单库读写、存储性能瓶颈，这时候就需要进行水平分库了，经过水平切分的优化，往往能解决单库存储量及性能瓶颈。但由于同一个表被分配在不同的数据库，需要额外进行数据操作的路由工作，因此大大提升了 系统复杂度 。

4.水平分表
水平分表是在同一个数据库内，把同一个表的数据按一定规则拆到多个表中。
它带来的提升是：
	优化单一表数据量过大而产生的性能问题；
	避免IO争抢并减少锁表的几率；
	库内的水平分表 ，解决了单一表数据量过大的问题，分出来的小表中只包含一部分数据，从而使得单个表的数据量变小，提高检索性能。

<table>
	<thead><tr><th>类型</th><th align="left">总结</th></tr></thead>
	<tbody><tr><td>垂直分表</td><td align="left">将 <strong>字段</strong> 按照使用 频率高低，是否超大文本类型，拆成多个表</td></tr>
		<tr><td>垂直分库</td><td align="left">专库专用。不同的库分配到不同的服务器上，如商品库，订单库，用户库等</td></tr>
		<tr><td>水平分库</td><td align="left">解决单表 <strong>数据行</strong> 过大问题，把一个库平移分成 <code>库1</code>、<code>库2</code>、… 、<code>库N</code></td></tr>
		<tr><td>水平分表</td><td align="left">解决单表 <strong>数据行</strong> 过大问题，把库内的同一个表分成 <code>表1</code>、<code>表2</code>、… 、 <code>表N</code></td></tr>
	</tbody></table>	