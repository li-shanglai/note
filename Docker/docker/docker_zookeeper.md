# Docker安装Zookeeper

```shell
#查询镜像
docker search zookeeper
#拉取镜像
docker pull zookeeper
#查看镜像
docker images
#安装、启动镜像
docker run --name zk -p 2181:2181 -d zookeeper
#查看当前启动的镜像
docker ps
#查看zookeeper日志
docker logs zk
#进入zookeeper容器
docker exec -it zk bash
#使用zkCli.sh开启客户端，连接zookeeper
zkCli.sh
#查看当前服务
ls /
```

