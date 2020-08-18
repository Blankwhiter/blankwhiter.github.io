#docker简易搭建Zookeeper集群

----------
写在前面：为什么要用zookeeper?主要是在分布式架构中，很多时候 需要考虑到数据的一致性问题，数据管理问题，以及事务的执行顺序等各种各样的问题，这时候就有一个zookeeper分布式服务器框架来帮我们解决这些问题。想想当你拥有成百上千台机器同步需要修改配置或者其他操作，会不会遇到这样的囧境。

docker zookeeper官网：[https://hub.docker.com/_/zookeeper/](https://hub.docker.com/_/zookeeper/)


## 第一步 创建docker网段
在centos窗口中，执行如下命令：
```bash
docker network create --subnet=172.20.0.0/16 net7
```
*注：172.20.0.0 网段（读者可以自定义自己所需的网段）
   16 子网掩码
   net7 网段名称 （读者可以自定义自己所需的网段名称）
说明：在此步创建网段，是为了合理规范便于治理，读者可自行选择是否创建*

## 第二步 创建Zookeeper集群
创建本地保存数据以及事务日志文件映射目录，在本文中将创建3个节点，故会创建对应3个节点的路径,在centos窗口中，执行如下命令创建所需目录：
```bash
cd /home/soft/
mkdir zookeepercluster
cd  zookeepercluster
mkdir zookeeper1DataDir
mkdir zookeeper1DataLogDir
mkdir zookeeper2DataDir
mkdir zookeeper2DataLogDir
mkdir zookeeper3DataDir
mkdir zookeeper3DataLogDir
```
*注：本文zookeeper集群存放的数据结构为/home/soft/zookeepercluster下 ，读者可自行选择*

创建完成后，接下来在centos窗口中，执行如下命令，拉取zookeeper镜像：
```bash
docker pull zookeeper
```
拉取完镜像后，创建zookeeper集群，在centos窗口中，执行如下命令：

```bash
docker run -d -v /home/soft/zookeepercluster/zookeeper1DataDir:/data -v /home/soft/zookeepercluster/zookeeper1DataLogDir:/datalog  -e ZOO_MY_ID=1 -e ZOO_SERVERS='server.1=172.20.0.2:2888:3888 server.2=172.20.0.3:2888:3888 server.3=172.20.0.4:2888:3888' --name=zookeeper1 --net=net7 --ip 172.20.0.2 --privileged zookeeper

docker run -d -v /home/soft/zookeepercluster/zookeeper2DataDir:/data -v /home/soft/zookeepercluster/zookeeper2DataLogDir:/datalog -e ZOO_MY_ID=2 -e ZOO_SERVERS='server.1=172.20.0.2:2888:3888 server.2=172.20.0.3:2888:3888 server.3=172.20.0.4:2888:3888' --name=zookeeper2 --net=net7 --ip 172.20.0.3 --privileged zookeeper

docker run -d -v /home/soft/zookeepercluster/zookeeper3DataDir:/data -v /home/soft/zookeepercluster/zookeeper3DataLogDir:/datalog  -e ZOO_MY_ID=3 -e ZOO_SERVERS='server.1=172.20.0.2:2888:3888 server.2=172.20.0.3:2888:3888 server.3=172.20.0.4:2888:3888' --name=zookeeper3 --net=net7 --ip 172.20.0.4 --privileged zookeeper
```
*注：‘ZOO_MY_ID=1’ 与 ‘server.1=172.20.0.2’ 的server.1对应，读者可自定义myid的编号
   2888 集群内机器通讯使用（Leader监听此端口） 
   3888 选举leader使用
说明：创建集群的时候请选择奇数个节点，以便节点选举leader*

## 第三步 验证是否成功搭建zookeeper集群
在centos窗口中，执行如下命令，进入容器中：
```bash
docker exec -it zookeeper1 bash
```
在容器中，执行如下命令可进行查看集群信息：
```bash
echo stat |nc 172.20.0.2 2181
```
其他两个容器，就不一一赘述。
下方是本人创建zookeeper集群全部的执行命令以及对应信息：
```bash
[root@localhost ~]# docker exec -it zookeeper1 bash
bash-4.4# echo stat | nc 172.20.0.2 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.20.0.2:42792[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 4
bash-4.4# exit
exit
[root@localhost ~]# docker exec -it zookeeper2 bash
bash-4.4# echo stat | nc 172.20.0.3 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.20.0.3:40206[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 4
Proposal sizes last/min/max: -1/-1/-1
bash-4.4# exit
exit
[root@localhost ~]# docker exec -it zookeeper3 bash
bash-4.4# echo stat | nc 172.20.0.4 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.20.0.4:43465[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: follower
Node count: 4
bash-4.4# 

```
在上方具体的信息可以看到是1个leader 2个follower。



对于zookeeper 3.4.x服务器版本，只有Curator 2.x.x才支持


写在最后，读者也可以参考dockerfile文件中的环境设置，通过（docker -v 参数）将本地服务器shitzookeeper配置映射到对应容器的zookeeper对应文件目录（/conf），详情请见附录








附录：
1.配置参数列表：
```bash
    ZOO_USER=zookeeper  #用户名
    ZOO_CONF_DIR=/conf  #配置文件路径 
    ZOO_DATA_DIR=/data  #数据文件路径
    ZOO_DATA_LOG_DIR=/datalog #事务日志文件路径  
    ZOO_PORT=2181             #客户端连接zookeeper端口
    ZOO_TICK_TIME=2000        #zookeeper独立工作时间单元 单位毫秒
    ZOO_INIT_LIMIT=5          #初始化连接时间 当初始化时间>2000*5（10秒）表示客户端连接失败
    ZOO_SYNC_LIMIT=2          #心跳时间 当间隔时间响应>2000*2(4秒) 表示节点失败
    ZOO_AUTOPURGE_PURGEINTERVAL=0  #清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能
    ZOO_AUTOPURGE_SNAPRETAINCOUNT=3 #这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
    ZOO_MAX_CLIENT_CNXNS=60 #单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制
```
2.four letter word(常用四字命令)
1.echo stat | nc localhost 2181 查看当前zookeeper状态信息
```bash
bash-4.4# echo stat | nc localhost 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /127.0.0.1:44935[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/552
Received: 1857
Sent: 1856
Connections: 1
Outstanding: 0
Zxid: 0x300000006
Mode: follower
Node count: 6

```
2.echo ruok | nc localhost 2181 查看当前zkserver是否启动，启动成功返回 imok
```bash
bash-4.4# echo ruok | nc localhost 2181
imok
```
3 echo dump | nc localhost 2181 列出未经处理的会话和临时节点
```bash
bash-4.4# echo dump | nc localhost 2181
SessionTracker dump:
org.apache.zookeeper.server.quorum.LearnerSessionTracker@499d64c5
ephemeral nodes dump:
Sessions with Ephemerals (0):

```
4 echo conf | nc localhost 2181 查看服务器配置
```bash
bash-4.4# echo conf | nc localhost 2181
clientPort=2181
dataDir=/data/version-2
dataLogDir=/datalog/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=1
initLimit=5
syncLimit=2
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0

```
5 echo cons | nc localhost 2181 展示连接到服务器的客户端信息
```bash
bash-4.4# echo cons | nc localhost 2181
 /127.0.0.1:55150[1](queued=0,recved=29,sent=29,sid=0x100046ccbd70000,lop=PING,est=1534428159463,to=30000,lcxid=0x1,lzxid=0xffffffffffffffff,lresp=93577238,llat=1,minlat=0,avglat=0,maxlat=13)
 /127.0.0.1:32959[0](queued=0,recved=1,sent=0)

```
6echo envi | nc localhost 2181 查看当前zookeeper环境变量
```bash
bash-4.4# echo envi | nc localhost 2181
Environment:
zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
host.name=e76b00d78b5e
java.version=1.8.0_171
java.vendor=Oracle Corporation
java.home=/usr/lib/jvm/java-1.8-openjdk/jre
java.class.path=/zookeeper-3.4.13/bin/../build/classes:/zookeeper-3.4.13/bin/../build/lib/*.jar:/zookeeper-3.4.13/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.13/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.13/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.13/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.13/bin/../zookeeper-3.4.13.jar:/zookeeper-3.4.13/bin/../src/java/lib/*.jar:/conf:
java.library.path=/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-862.3.3.el7.x86_64
user.name=zookeeper
user.home=/home/zookeeper
user.dir=/zookeeper-3.4.13

```
7 echo mntr | nc localhost 2181 监控zk健康信息
```bash
bash-4.4# echo mntr | nc localhost 2181
zk_version	3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
zk_avg_latency	0
zk_max_latency	552
zk_min_latency	0
zk_packets_received	1911
zk_packets_sent	1910
zk_num_alive_connections	2
zk_outstanding_requests	0
zk_server_state	follower
zk_znode_count	6
zk_watch_count	0
zk_ephemerals_count	0
zk_approximate_data_size	63
zk_open_file_descriptor_count	32
zk_max_file_descriptor_count	65536
zk_fsync_threshold_exceed_count	0
```
8 echo wchs | nc localhost 2181 展示watch的信息
```bash
bash-4.4# echo wchs | nc localhost 2181
0 connections watching 0 paths
Total watches:0

```
9 wchc wchp session与watch以及path与watch信息
使用wchc wchp命令，需要在zookeeper的配置文件加入41w.commadns.whitelist=* .完成白名单的配置，并重启zookeeper服务。