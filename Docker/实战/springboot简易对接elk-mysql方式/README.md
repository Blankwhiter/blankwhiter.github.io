
# springboot简易对接elk-mysql方法

# 一、lasticsearch单例环境准备

### 1.1 在centos中 创建对应映射目录 /home/software/elasticsearch/data,以及编写/home/software/elasticsearch/config/下es-single.yml，内容如下
```xml
network.bind_host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```
*注：是为了解决其实地址可以访问，以及跨域问题*
### 1.2 在centos中 执行如下命令搭建elasticsearch单例实例
```bash
 chmod 777 /home/software/elasticsearch/data
 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9210:9200 -p 9310:9300  -e "discovery.type=single-node" -v /home/software/elasticsearch/data:/usr/share/elasticsearch/data -v /home/software/elasticsearch/config/es-single.yml:/usr/share/elasticsearch/config/elasticsearch.yml --name es-single elasticsearch:7.9.3
```

# 二、logstash环境准备

### 2.1 将mysql-connector-java-8.0.19.jar放置于`centos的/home/software/pipeline/`
### 2.2 在centos的/home/software/pipeline/下 创建pipeline.conf，文件内容如下： 
```xml
input {
  jdbc {
    # 驱动包位置
    jdbc_driver_library => "/usr/share/logstash/pipeline/mysql-connector-java-8.0.19.jar"
    # 驱动
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    # 数据库地址
    jdbc_connection_string => "jdbc:mysql://47.99.200.71:3306/test?characterEncoding=UTF-8&useSSL=false&autoReconnect=true"
    # 数据库连接用户名
    jdbc_user => "root"
    # 数据库连接用户密码
    jdbc_password => "xxxxxxxxxx"
    # 执行sql语句文件位置
    # statement_filepath => "filename.sql"
    # tracking_column => create_time
    # record_last_run => true
    # 执行sql
    statement => "SELECT * from schedule_job_log  "
    # 是否分页
    jdbc_paging_enabled => "true"
    # 分页数量
    jdbc_page_size => "50000"
    type => "jdbc"
    use_column_value => false
    last_run_metadata_path => "/usr/share/logstash/pipeline/metadata/last_run.txt"
    # 执行任务时间间隔，各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
    schedule => "* * * * *"
  }
}
 
filter {
	json {
		source => "message"
		remove_field => ["message"]
	}
}
 
output {
  elasticsearch {
        hosts => "47.99.200.71:9200"
        index => "test-mysql"
        document_type => "job_log"
        # 数据库中的id
        document_id => "%{log_id}"
  }
 }
```
*注：这里定义两个输入两个输出，通过add_field自定义属性 用于输出判断*


```
字段介绍：

    input.jdbc.jdbc_driver_library  jdbc驱动的位置
    input.jdbc.jdbc_driver_class    驱动类名
    input.jdbc.jdbc_connection_string   数据库连接字符串
    input.jdbc.jdbc_user    用户名
    input.jdbc.jdbc_password  密码
    input.jdbc.schedule   更新计划(参考linux crontab)
    input.jdbc.jdbc_default_timezone   时区，默认没有时区，日志里时间差8小时，中国需要用Asia/Shanghai
    input.jdbc.statement_filepath 导出数据的sql文件，就是上面写的
    input.jdbc.use_column_value  如果是true,sql_last_value是tracking_column指定字段的数字值，false就是时间，默认是false
    input.jdbc.last_run_metadata_path  保存sql_last_value值文件的位置
    output.elasticsearch.hosts elasticsearch服务器，填多个，请求会负载均衡。
    output.elasticsearch.index 索引名
    output.elasticsearch.document_id   生成文件的id,这里使用sql产生的car_id
    output.elasticsearch.document_type  文档类型 
    output.stdout 配置的是命令行输出，使用json
```






### 2.3 在centos的/home/software/logstash/下 创建logstash.yml，文件内容如下：
```xml
xpack：
  monitoring：
    enabled：true
```



### 2.4 调整-Xms256m  -Xmx256m大小：`/home/software/config/jvm.options`
```
## JVM configuration

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms256m
-Xmx256m

################################################################
## Expert settings
################################################################
##
## All settings below this section are considered
## expert settings. Don't tamper with them unless
## you understand what you are doing
##
################################################################

## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly

## Locale
# Set the locale language
#-Duser.language=en

# Set the locale country
#-Duser.country=US

# Set the locale variant, if any
#-Duser.variant=

## basic

# set the I/O temp directory
#-Djava.io.tmpdir=$HOME

# set to headless, just in case
-Djava.awt.headless=true

# ensure UTF-8 encoding by default (e.g. filenames)
-Dfile.encoding=UTF-8

# use our provided JNA always versus the system one
#-Djna.nosys=true

# Turn on JRuby invokedynamic
-Djruby.compile.invokedynamic=true
# Force Compilation
-Djruby.jit.threshold=0
# Make sure joni regexp interruptability is enabled
-Djruby.regexp.interruptible=true

## heap dumps

# generate a heap dump when an allocation from the Java heap fails
# heap dumps are created in the working directory of the JVM
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps
# ensure the directory exists and has sufficient space
#-XX:HeapDumpPath=${LOGSTASH_HOME}/heapdump.hprof

## GC logging
#-XX:+PrintGCDetails
#-XX:+PrintGCTimeStamps
#-XX:+PrintGCDateStamps
#-XX:+PrintClassHistogram
#-XX:+PrintTenuringDistribution
#-XX:+PrintGCApplicationStoppedTime

# log GC status to a file with time stamps
# ensure the directory exists
#-Xloggc:${LS_GC_LOG_FILE}

# Entropy source for randomness
-Djava.security.egd=file:/dev/urandom

# Copy the logging context from parent threads to children
-Dlog4j2.isThreadContextMapInheritable=true

```

1.ES中8小时时差的问题？
解决方法：从源头解决问题
在jdbc.conf配置文件中只要是有关时间的字段都手动+8小时

```
　filter {
      json {
          source => "message"
          remove_field => ["message"]
      }
  　　 // date类型不能省略，不然会报错，       就是把当前字段+8小时后赋值给新的字段，然后再取新字段的值赋值给老的字段，再把新的字段删除
      date {
        match => ["message","UNIX_MS"]
        target => "@timestamp"
         }
           ruby {
                  code => "event.set('timestamp', event.get('@timestamp').time.localtime + 8*60*60)"
           }      
          ruby{   
                  code => "event.set('@timestamp',event.get('timestamp'))"
          }       
          mutate{ 
                 remove_field => ["timestamp"]
          }      
          
     date {
      match => ["message","UNIX_MS"]
      target => "create_time"
           } 
           ruby {
                   code => "event.set('@create_time', event.get('create_time').time.localtime + 8*60*60)"
           }       
          ruby {   
                   code => "event.set('create_time',event.get('@create_time'))"
           }       
          mutate { 
           remove_field => ["@create_time"]
          }
          
          date {
          match => ["message","UNIX_MS"]
          target => "update_time"
           }
           ruby {
                   code => "event.set('@update_time', event.get('update_time').time.localtime + 8*60*60)"
           }       
          ruby {   
                   code => "event.set('update_time',event.get('@update_time'))"
           }       
          mutate {
           remove_field => ["@update_time"]
          } 
    }
```




### 2.5 在centos中 执行如下命令搭建logstash单例环境
```bash
docker run -p 5000:5000 -p 5001:5001 -p 9600:9600 -d --privileged -e xpack.monitoring.elasticsearch.url=http://47.99.200.71:9200  -v /home/software/config/jvm.options:/usr/share/logstash/config/jvm.options    -v /home/software/logstash/logstash.yml:/etc/logstash/logstash.yml  -v /home/software/pipeline:/usr/share/logstash/pipeline/ --name logstash-single  logstash:7.9.3 -f /usr/share/logstash/pipeline/pipeline.conf

```



附录：
(logstash版本7以下需要安装jdbc插件)进入容器安装`logstash-input-jdbc`
```
./logstash-plugin install logstash-input-jdbc
```





