# docker简易实现热备份数据库数据

写在前面：冷备份是关闭数据库时候的备份方式，通常做法是拷贝数据文件。但大型网站无法做到关闭业务备份数据，所以冷备份是不是最佳选择，故选择热备份。当然通过pxc集群，读者可以先关闭其中一个节点进行备份，而后重新开启加入集群，自然集群数据再次进行同步。MySQL常见的热备份有LVM和XtraBackup两种。但由于当LVM在备份时锁表只能读取不能写入。
XtraBackup备份不锁表，不会打断正在执行的食物，能够压缩等功能节约磁盘空间和流量。可采取增量备份、全量备份。

## 第一步 创建docker卷
在centos窗口中，执行如下命令：
```bash
docker volume create backup
```
## 第二部 映射docker卷
这里选取第一个节点 进行数据备份，读者可自行选择节点将备份卷映射到容器中重新创建即可。
首先将node1节点停止 删除。由于一开始并没有将备份的docker卷映射其中，故重新创建改node1容器。
在centos窗口中，执行如下命令：
```bash
docker stop node1
docker rm node1
docker run -d -p 3306:3306 -v v1:/var/lib/mysql -v backup:/data -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456  -e CLUSTER_JOIN=node1  --privileged --name=node1 --net=net2 --ip 172.18.0.2 pxc
```
注： -v backup:/data | 将备份docker卷进行映射
     -e CLUSTER_JOIN=node2 | 选择同步的节点，选择其他某个节点进行数据同步

## 第三步 PXC全量备份
在PXC容器中安装XtraBackup，并执行备份。在pxc容器node1中，执行如下命令：
```bash
apt-get update
apt-get install percona-xtrabackup-24
innobackupex --user=root --password=123456 /data/backup/full
```
执行后进行对应目录进行查看，如下所示：
```bash
root@6596a013857c:/# cd /data/backup/full/2018-07-02_06-32-52/
root@6596a013857c:/data/backup/full/2018-07-02_06-32-52# ls
backup-my.cnf  ib_buffer_pool  ibdata1	mysql  performance_schema  sys	xtrabackup_binlog_info	xtrabackup_checkpoints	xtrabackup_info  xtrabackup_logfile

```
## 第四步 验证是否全量备份成功，进行PXC全量恢复
数据库可以热备份，但不能热还原。为了避免恢复过程中的数据同步，我们采用空白的MySQL还原数据，然后再创建PXC集群。请读者自行实现。这里我们主要实现冷备份数据。
首先先停止其余节点容器（node2,node3,node4）并删除容器以及对应的docker卷。在centos窗口中，执行如下命令：
```bash
docker stop node2  node3   node4
docker rm node2  node3   node4
docker volume rm v2 v3 v4
```
进入node1容器中，删除数据，回滚事务，恢复数据，重启数据库服务。执行如下命令：
```bash
docker exec -it node1 bash
rm -rf /var/lib/mysql/*
innobackupex --user=root --password=abc123456 --apply-back /data/backup/full/2018-07-02_07-29-54/
innobackupex --user=root --password=123456 --copy-back /data/backup/full/2018-07-02_07-29-54/
service mysql stop
service mysql start
```
*注：还原数据前要将未提交的事务进行回滚，还原数据之后重启MySQL*
## 第五步 连接到node1数据库,查看数据是否恢复。
数据库连接信息如下：
<pre>
数据库ip地址（192.168.9.144 ）---宿主机ip
端口 3306
用户 root
密码 123456
</pre>
连接上数据库查看数据是否还原成功。成功后重新建立集群，具体步骤请参考《docker简易搭建MySQL集群》*[https://blog.csdn.net/belonghuang157405/article/details/80774506](https://blog.csdn.net/belonghuang157405/article/details/80774506 "docker简易搭建MySQL集群")。



