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
```
