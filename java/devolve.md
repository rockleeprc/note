# 环境
## 测试环境
### nacos
* http://39.107.64.247:8848/nacos
    * nacos:nacos
* lb
    * 172.17.97.220:8848
### gateway
* lb
    * 172.17.97.220:8000
    * curl -X GET "http://172.17.97.220:8000/api/stmt/test/info" -H  "accept: */*"
### impala
* lb
    * 172.17.97.220:21050
    * 39.96.216.143:21060
# jenkins


## 预生产环境
### nacos
* http://47.94.169.45:8848/nacos/#/login
    * nacos:nacos
* lb
    * 10.1.5.185:8848
* 部署
    * 10.1.5.187:8848
    * 10.1.5.188:8848
    * 10.1.5.189:8848

### gateway
* lb
    * 10.1.5.185:8000
    * curl -X GET "http://10.1.5.185:8000/api/stmt/test/info" -H  "accept: */*"
### impala
* lb
    * 10.1.5.185:21050
    * 10.1.5.185:21000

## 生产
### nacos
* http://123.57.230.127:8848/nacos
    * nacos:nacos
* lb
    * 172.18.10.234:8848
* 部署
    * 172.19.10.22:8848
    * 172.19.10.23:8848
    * 172.19.10.24:8848

### gateway
* lb
    * 172.16.10.67:8000
    * curl -X GET "http://172.16.10.67:8000/api/stmt/test/info" -H  "accept: */*"
    * curl -X GET "http://172.17.0.2:9900/actuator/metrics" -H  "accept: */*"
    * curl -X GET "http://172.17.0.2:9900/actuator/prometheus" -H  "accept: */*"
### imapal
* lb
    * 172.16.10.67:21050
    * 172.16.10.67:21000

# 部署
```shell
# 切换root用户
sudo -i

# 查看镜像
docker images

#connect docker hub
docker login --username=kd_wangluo --password='fast@123!QW#4' registry-vpc.cn-beijing.aliyuncs.com

# download images
docker pull registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-agent-service:test
docker pull registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-gateway-service:test

docker pull registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred
docker pull registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:pred

docker pull registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:prod
docker pull registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:prod

# 查看容器
docker ps -a

# 获取容器ip 
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' a67eef8a05c3

# nacos 下线服务

# 停止运行的容器
docker stop bbd5a95346b5

# 删除容器实例
docker rm cc768bd3a02e


# jmap可以使用（启动容器时配置）
--cap-add=SYS_PTRACE 
--security-opt seccomp:unconfined

# agent启动容器实例
## test
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-1 -p 9900:9900 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-agent-service:test
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-2 -p 9800:9900 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-agent-service:test

## pred
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-1 -p 9900:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-2 -p 9800:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred

## prod
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-1 -p 9900:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:prod
docker run --cap-add=SYS_PTRACE --name wisdomshare-agent-service-2 -p 9800:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:prod

# gatway启动容器实例
## test
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-1 -p 8000:8000 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-gateway-service:test
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-2 -p 8100:8000 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-gateway-service:test

## pred
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-1 -p 8000:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:pred
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-2 -p 8100:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:pred

## prod
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-1 -p 8000:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:prod
docker run --cap-add=SYS_PTRACE --name wisdomshare-gateway-service-2 -p 8100:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:prod
```

#! /bin/bash

#connect docker hub
docker login --username=kuaidao2020 --password='fast@123!QW#4' registry.cn-beijing.aliyuncs.com
#download images docker pull registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:prod

#create ins1
docker run --name wisdomshare-agent-service-1 -p 9900:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-serv
ice:prod

#create ins2
docker run --name wisdomshare-agent-service-2 -p 9800:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-serv
ice:prod

#gateway
docker run --name wisdomshare-gateway-service-1 -p 8000:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-
service:prod

docker run --name wisdomshare-gateway-service-2 -p 8100:8000 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-
service:prod

#获取容器ip
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cc768bd3a02e

#查看容器
docker ps -a

#删除容器
docker rm cc768bd3a02e