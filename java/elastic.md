## 一、ES基本概念

### es结构

| RDBMS          | ElasticSearch |
| -------------- | ------------- |
| 数据库Database | 索引Index     |
| 表Table        | 类型Type      |
| 数据行Row      | 文档Document  |
| 数据列Column   | 字段Field     |

| Elasticsearch         | MySQL    |
| --------------------- | -------- |
| index（索引，名词）   | database |
| doc type（文档类型）  | table    |
| document（文档）      | row      |
| field（字段）         | column   |
| mapping（映射）       | schema   |
| query DSL（查询语言） | SQL      |

### es概念

* cluster
* node
* shard
* replia

### ES2.X文件系统存储类型

* Simple FS（简单文件系统）
* NIO FS（NIO文件系统）
* MMap FS（内存映射文件系统）
* Hybrid MMap / NIO FS（缺省值, default_fs）



## 二、install

### jdk

* elastic需要jdk，5.x需要jdk1.8，不要使用jdk1.9

### 用户/组

- ES无法使用root用户启动，单独创建用户/组

```shell
groupadd elsearch # 创建组
useradd elsearch -g elsearch # 创建用户，并分配组
chown -R elsearch:elsearch elasticsearch-5.6.3  # 该命令是更改该文件夹下所属的用户组的权限
```

### 下载

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.10.tar.gz
sha1sum elasticsearch-5.6.10.tar.gz 
tar -xzf elasticsearch-5.6.10.tar.gz
cd elasticsearch-5.6.10/ 
```

### 修改配置

* `elasticsearch-5.6.10\config\elasticsearch.yml`

```yaml
cluster.name: ${my-application}
node.name:    ${HOSTNAME}
path.data: /path/to/data
path.logs: /path/to/logs
network.host: ${ES_NETWORK_HOST}
http.port: 9200
```

* 创建两个数据目录path.data、path.logs

### 运行

```shell
./elasticsearch -d
```

### 检查

```shell
http://localhost:9200 # elastic默认工作在9200端口
```

## 内存优化

```shell
ps -ef|grep elasticsearch
export ES_HEAP_SIZE=1g # 设置环境变量
```



### 详细内容

* [elastic5.6 官方安装文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/zip-targz.html)
* [ES5.4.0安装包下载地址](https://blog.csdn.net/laoyang360/article/details/73368740)

## 可视化插件

### head

* elasticsearch-head是一个用来浏览、监控Elastic Search状态、与Elastic Search簇进行交互的web前端展示插件 

* [head插件git地址](https://github.com/mobz/elasticsearch-head)

* head插件需要node.js

```shell
yum install -y nodejs
```

* head 插件的执行文件是有grunt 命令来执行的 
* 访问9100端口有问题时，修改elastic配置文件elasticsearch.yml，增加如下配置

```shell
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### IK粉刺

[github地址](https://github.com/medcl/elasticsearch-analysis-ik)

## 三、ES Restful API 

* 1）GET：获取请求对象的当前状态。
* 2）POST：改变对象的当前状态。  
* 3）PUT：创建一个对象。  
* 4）DELETE：销毁对象。 
* 5）HEAD：请求获取对象的基础信息 



## 四、Springboot集成

* spring-data配置application.yml 

```yaml
spring:
  data:
    elasticsearch:
      cluster-name: my-application
      cluster-nodes: 服务器Ip:9300
```

* jest配置application.yml 

```
spring:
  elasticsearch:
    jest:
      uris: http://123.206.46.157:9200
      username: elastic
      password: changeme
      read-timeout: 5000
      connection-timeout: 5000
```



## 五、MySQL与ES数据同步

* 数据采集->MySQL->ES

### delete

* mysql中数据不删除，通过标志位逻辑删除，将状态同步到ES

### insert,update

*  logstash-input-jdbc 插件同步数据

[logstash-input-jdbc](https://blog.csdn.net/laoyang360/article/details/51747266)



## 六、elastic客户端

* Node Client：废弃
* Transport Client：7.x不支持
* Rest API：



## 参考

* [史上超全面的Elasticsearch使用指南](https://blog.csdn.net/yueshutong123/article/details/80956223)
* [ ElasticSearch使用](https://blog.csdn.net/The_lone_wolfs/article/details/79530580)
* [ElasticSearch使用总结](https://blog.csdn.net/Mrxuchen/article/details/80028386)
* [23 个很有用的 ElasticSearch 查询示例](http://coyee.com/article/10764-23-useful-elasticsearch-example-queries/)
* [死磕 Elasticsearch 方法论](https://blog.csdn.net/laoyang360/article/details/79293493)
* [elastic知识库](http://lib.csdn.net/wojiushiwo987/structure/deep_elasticsearch)
* [Springboot1.x整合elastic5.x](https://blog.csdn.net/come_sky/article/details/80291108)

