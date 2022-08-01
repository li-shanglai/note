# Docker安装Nacos

```shell
docker search nacos

#拉取镜像
docker pull nacos/nacos-server
#创建映射文件
mkdir -p /root/nacos/init.d /root/nacos/logs
touch /root/nacos/init.d/custom.properties
#文件中写入配置
echo 'management.endpoints.web.exposure.include=*' > /root/nacos/init.d/custom.properties
#创建容器
docker run -d -p 8848:8848 -e MODE=standalone -e PREFER_HOST_MODE=hostname -v /root/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties -v /root/nacos/logs:/home/nacos/logs --restart always --name nacos nacos/nacos-server
#启动
docker start nacos
```

