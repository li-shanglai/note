# Docker安装Consul

```shell
# 设置yum源
yum install -y yum-utils device-mapper-persistent-data lvm2 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast

# 安装Docker-CE
yum -y install docker-ce

#设置开机自启动
systemctl enable docker 
#搜索
docker search consul
#拉取
docker pull consul
#启动
docker run --name consul1 -d -p 8500:8500 consul
```

