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

# jenkins

http://jenkins.kuaidaoapp.com
hx:hx2021

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
    * 
    * curl -X GET "http://10.1.5.185:8000/api/stmt/test/info" -H  "accept: */*"
### impala
* lb
    * 10.1.5.185:21050
    * 10.1.5.185:21000


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

registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred
registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-gateway-service:pred

# 查看容器
docker ps -a

# 获取容器ip 
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' f4ef91888877

# nacos 下线服务

# 停止运行的容器
docker stop bbd5a95346b5

# 删除容器实例
docker rm cc768bd3a02e

# 启动容器实例
docker run --name wisdomshare-agent-service-1 -p 9900:9900 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-agent-service:test

docker run --name wisdomshare-agent-service-1 -p 9900:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred

# jmap可以使用
--cap-add=SYS_PTRACE 
--security-opt seccomp:unconfined

# 启动容器实例
docker run --name wisdomshare-agent-service-2 -p 9800:9900 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-agent-service:test

docker run --name wisdomshare-agent-service-2 -p 9800:9900 -d registry.cn-beijing.aliyuncs.com/kdhub/huixiang-wisdomshare-agent-service:pred

# 启动容器实例
docker run --name wisdomshare-gateway-service-1 -p 8000:8000 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-
gateway-service:test

# 启动容器实例
docker run --name wisdomshare-gateway-service-2 -p 8100:8000 -d registry-vpc.cn-beijing.aliyuncs.com/fasttrack/huixiang-wisdomshare-
gateway-service:test

```

