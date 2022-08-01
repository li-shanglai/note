# Docker安装Sentinel

```shell
docker pull bladex/sentinel-dashboard
#运行,需要映射两个端口号
docker run --name sentinel -d -p 8858:8858 -p 8858:8858 -d bladex/sentinel-dashboard
```

```yaml
#使用docker部署 需要在yml中配置client-ip: 本机ip
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        # Nacos服务注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport:
        # sentinel dashboard 地址
        dashboard: localhost:8858
        # 默认为8719，如果被占用会自动+1，直到找到为止
        port: 8719
        # 本地机器ip
        client-ip: 本机ip
```

