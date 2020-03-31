### 操作系统版本
uname -r 内核
lsb_release -a 操作系统版本
cat /etc/redhat-release 操作系统版本

### 三要素
镜像：
容器：
仓库：

### 安装
* 之前安装的需要先卸载
```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

$ sudo yum-config-manager --enable docker-ce-nightly
$ sudo yum-config-manager --enable docker-ce-test
$ sudo yum install docker-ce docker-ce-cli containerd.io
$ sudo systemctl start docker
```

```shell
# 本地镜像模版
docker images 
# 镜像大小
docker system df
# 删除镜像
docker rmi tomcat
# 启动tomcat 【宿主机端口（物理机）:容器端口（docker tomcat）】
docker run -p 8080:8080 --name tomcat -d tomcat
# 查看进程
docker ps
# 关闭容器
docker stop containerID
# 强制删除正在运行的容器
docker rm -f 7285741ac0db
# 进入到容器
docker exec -it 7285741ac0db /bin/bash
```


