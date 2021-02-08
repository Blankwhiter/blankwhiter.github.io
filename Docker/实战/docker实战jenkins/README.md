# docker实战jenkins


1.安装jenkins环境
```
docker run --name jenkins -u root -d -p 8080:8080  -p 8088:8088 -p 50000:50000 -v /home/blueocean/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean

```
参数说明：
```
-u root：以 root 权限启动，防止出现权限问题
-p 8080:8080：端口映射，服务器的 7005 端口映射容器的 8080 端口
-p 50000:50000：Jenkins代理默认通过TCP端口50000与Jenkins主机通信
-v /home/blueocean/jenkins:/var/jenkins_home：把容器内的 Jenkins 目录挂载到服务器的 /data/jenkins 目录以防容器没了，数据也没了
-v /var/run/docker.sock:/var/run/docker.sock：保证容器内的 docker 与 服务器上 docker 的通讯
-p 8088:8088 是当部署服务时候所需要的暴露的端口
```

直接通过容器日志可以看到密码：docker logs xxxx（容器 ID），登录后 推荐插件安装
 
2.jenkins 全局工具配置
2.1进入到 jenkins 容器中 echo $JAVA_HOME 获取 java 环境安装地址
```
echo $JAVA_HOME
/usr/lib/jvm/java-1.8-openjdk
```
2.2获得git安装路径
```
apk info -L git
```
 

3.安装 Maven Integration 插件（构建maven项目选项）
在插件界面点击Available(可选插件)，然后在右边Filter搜索需要的插件（ Maven Integration）
安装后重启


4.通过系统管理的全局设置，需要设置jdk git mvn环境
系统管理–>全局工具配置
```
JDK => JAVA_HOME : /usr/lib/jvm/java-1.8-openjdk
GIT => git: usr/bin/git
MAVEN => 新增maven
```


5.创建项目（git项目本地部署）
5.1源码管理
Git Repository URL : https://github.com/Blankwhiter/test-jenkins.git
分支：*/main

5.2构建触发器
轮询SCM
日程表：*/1 * * * *
*/1 * * * *(每分钟执行一次)


5.3构建
Root POM： pom.xml
Goals and options: clean install -Dmaven.test.skip=true

5.4Post Steps
Run only if build succeeds
增加构建步骤：选择 执行Shell

```
#!/bin/bash
#docker里集成jenkins自动化部署springboot
#服务名称
SERVER_NAME=test-jenkins
#源JAR/WAR名称
JAR_NAME=test-jenkins-0.0.1-SNAPSHOT
#打包目录
JAR_PATH=/var/jenkins_home/workspace/test-maven/target/
# 打包完成之后，把 jar 包移动到运行 jar 包的目录 ---> word_daemon, work_daemon 这个目录需要自己提前创建
JAR_WORK_PATH=/var/jenkins_home/workspace/test-jenkins/target

echo "查询进程ID-->$SERVER_NAME"
PID=`ps -ef | grep "$SERVER_NAME" | awk "{print $2}"`
echo "得到进程ID-->$PID"
echo "结束进程开始"
#for id in $PID
#do
#    kill -9 $id
#    echo "结束进程ID-->$id"
#done
kill -9 $PID
echo "结束进程结束"

echo "复制jar包到执行目录：cp $JAR_PATH/$JAR_NAME.jar $JAR_WORK_PATH"
mkdir -p $JAR_WORK_PATH
cp $JAR_PATH/$JAR_NAME.jar $JAR_WORK_PATH
echo "复制JAR包成"
cd $JAR_WORK_PATH
chmod 755 $JAR_NAME.jar
BUILD_ID=dontKillMe nohup java -jar $JAR_NAME.jar &
```



附录：
1.如果在创建项目时候，没有“创建一个Maven 项目”的选项。
 点击“可选插件”  然后在右边的过滤输入框中输入搜索关键字： Maven Integration  或者 Pipeline Maven Integration Plugin ，搜索到了以后，点击直接安装，安装完成后重启就好了。

3.在调试项目的接口自动化工程的时候，遇到一个分支问题，如下
ERROR: Couldn't find any revision to build. Verify the repository and branch configuration for this job.
是jenkins找不到分支来拉指定的git代码，是因为github上master节点名称变更为main

3.重载Jenkins配置信息
http://localhost:8080/reload

4.停止Jenkins服务器
http://localhost:8080/exit

5.重启Jenkins
http://localhost:8080/restart


6.开启邮箱通知
Jenkins Location
   系统管理员邮件地址 ： belonghuang@126.comc
邮件通知
  SMTP： smtp.126.com
  用户默认邮件后缀： @126.com
  用户名： belonghuang@126.com
  密码： EOWSDJSDWWRDFDRW
  SMTP端口： 25


7.安装Publish Over SSH插件
系统配置
  Publish over SSH
   Name： 192.168.0.33
   Hostname：192.168.0.33
   Username： root
   RemoteDiretory： /home

SSH配置
Name ： 这里是自定义的ssh远程服务器，如果有多台,点击add post-build step 继续添加
Source files ： 源文件地址，相对地址
Remove prefix ： 忽略前缀路径
Remote directory ： 远程服务器要保存的文件目录(会基于系统配置的RemoteDiretory，如果填写/, 项目配置时可以使用绝对路径)
Exec command ： 直接写shell脚本，或远程服务器的脚本名称

Source files必须配置为jar包的相对地址，此相对地址即workspace后面的地址（不包含workspace）
例如：目标文件路径是/var/jenkins_home/workspace/test-jenkins/target/xxx-web-1.1.jar
那么你的
Source files的值就是target/test-jenkins-0.0.1-SNAPSHOT.jar
Remove prefix就是target/
Remote directory的值取决于Jenkins -> configuration -> Publish over SSH设置中的Remote Directory参数

