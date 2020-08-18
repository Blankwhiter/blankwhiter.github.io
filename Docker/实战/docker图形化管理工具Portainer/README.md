# docker图形化管理工具Portainer 

>Portainer是一款轻量级的图形化管理工具，通过它我们可以轻松管理不同的docker环境。Portainer部署和使用都非常的简单，它由一个可以运行在任何docker引擎上的容器组成。Portainer提供管理docker的containers、images、volumes、networks等等。它兼容独立的docker环境和swarm集群模式。基本满足中小型单位对docker容器的管理工作。

演示网址：http://demo.portainer.io 账号admin 密码 tryportainer （读者可先自行体验）

# 第一步 拉取Portainer镜像,以及运行容器
在centos窗口中，执行如下命令拉取镜像，以及运行容器：
```bash
 docker pull portainer/portainer
 docker volume create portainer_data
 docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

```

# 第二步 验证是否成功
在浏览器中访问 http://ip:9000

更多具体操作，请参考 http://blog.51cto.com/bovin/2170723