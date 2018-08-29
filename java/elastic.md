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

### 可能遇到的问题

* 不能使用root启动，新建elsearch用户和组

```shell
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
```

* 日志目录权限不够，日志目录也要修改用户和组权限

```shell
2018-08-25 12:07:39,060 main ERROR RollingFileManager (/data/es/logs/my-application.log) java.io.FileNotFoundException: /data/es/logs/my-application.log (权限不够) java.io.FileNotFoundException: /data/es/logs/my-application.log (权限不够)
```

* 修改/etc/security/limits.conf 文件

```shell
elsearch soft memlock unlimited
elsearch hard memlock unlimited

* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

* 修改/etc/sysctl.conf 文件

```shell
vm.max_map_count=655360
$ sysctl -p
```

* 关闭防火墙

```shell
$ firewall-cmd --state # 查看防火墙状态
$ systemctl stop firewalld # 临时关闭
$ systemctl disable firewalld # 禁止开机启动
```

* 运行

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

* 下载插件

```shell
$ wget https://github.com/mobz/elasticsearch-head/archive/master.zip
```

* 解压，如果没有unzip需要安装

```shell
yum install zip unzip
```

* 安装nodejs(centos)

```shell
curl -sL https://rpm.nodesource.com/setup_8.x | bash - 
yum install -y nodejs
[root@node01 local]# node -v
v8.11.4
[root@node01 local]# npm -v
5.6.0
```

* 安装nodejs(ubuntu)

```shell
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

* 安装grunt

```shell
npm install grunt --save-dev
[root@node01 elasticsearch-head-master]# npm install
```

* 修改elasticsearch.yml配置文件，增加如下配置

```SHELL
http.cors.enabled: true
http.cors.allow-origin: "*"
```

* 修改/usr/local/elasticsearch-head-master/Gruntfile.js，增加hostname配置

```shell
       connect: {
            server: {
                options: {
                    port: 9100,
                    hostname: '*',
                    base: '.',
                    keepalive: true
                }
            }
        }
```

* 修改elasticsearch-head-master/_site/app.js，ip地址修改为本机

```
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://47.106.214.111:9200";
```

* 启动

```shell
[root@node01 bin]# ./grunt server &
[1] 1678
[root@node01 bin]# (node:1678) ExperimentalWarning: The http2 module is an experimental API.
Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100
```

* 访问

```shell
http://192.168.56.11:9100/
```





* 创建mapping

```shell
# 1、先创建所索引
# 2、创建映射
put http://47.106.214.111:9200/map/_mapping/articles2/
put http://47.106.214.111:9200/map/articles3/_mapping/
```

* mapping

```json
{
	"settings": {
		"number_of_shards": 5,
		"number_of_replicas": 1,
		"analysis": {
			"analyzer": { //创建索引时指定分词器
				"ik": {
					"tokenizer": "ik_max_word"
				}
			}
		}
	},
	"mappings": { //没有创建索引时指定，如果创建索引后执行：index_already_exists_exception
		"person": { //type名称
			"_all": {
				"enabled": false
			},
			"properties": { //指定文档中字段类型
				"id": {
					"type": "integer"
				},
				"name": {
					"type": "text",
					"analyzer": "ik_max_word",
					"search_analyzer": "ik_max_word"
				},
				"birth": {
					"type": "date",
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
				},
				"status": {
					"type": "keyword"
				}
			}
		}
	}
}
```

* 对分词器的测试

```shell
http://47.106.214.111:9200/_analyze?pretty&analyzer=ik_max_word&text=这是一个对分词器的测试
```







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

### IK分词

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



## logstash 安装

* 下载地址

```shell
https://artifacts.elastic.co/downloads/logstash/logstash-5.6.10.tar.gz	
```

* ruby开发，需要gem命令，替换国内镜像

```shell
# 安装gem
yum install gem
# 替换镜像
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
# 查看镜像地址
gem sources -l
```

* 测试logstash是否安装成功

```shell
cd bin
./logstash -e 'input { stdin { } } output { stdout {} }'
# 随便输入
hello word
{
      "@version" => "1",
          "host" => "iZwz9j3wje0m1kqkiqlyb0Z",
    "@timestamp" => 2018-08-27T09:01:42.289Z,
       "message" => "hello word"
}
```

* 修改/usr/local/logstash-5.6.10/Gemfile文件

```shell
source "https://gems.ruby-china.com"
```

* 安装logstash-input-jdbc（过程很漫长）

```shell
cd bin
root@iZwz9j3wje0m1kqkiqlyb0Z:/usr/local/logstash-5.6.10/bin# ./logstash-plugin install logstash-input-jdbc
Validating logstash-input-jdbc
Installing logstash-input-jdbc
Installation successful
```

* 配置mysql.conf

```shell
input {
    stdin {
    }
    jdbc {
      # 数据库
      jdbc_connection_string => "jdbc:mysql://localhost:3306/test01"
      # 用户名密码(你的数据库的用户名和密码)
      jdbc_user => "root"
      jdbc_password => "password"
      # jar包的位置
      jdbc_driver_library => "/usr/local/logstash-5.6.3/mysql-connector-java-5.1.41.jar"
      # mysql的Driver
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      #处理中文乱码问题
      codec => plain { charset => "UTF-8"}
      # true使用其它字段追踪，false使用时间字段追踪
      use_column_value => false
      tracking_column => birth
      record_last_run => true
     #上一个sql_last_value值的存放文件路径, 必须要在文件中指定字段的初始值
     #last_run_metadata_path => "G:\Developer\Elasticsearch5.5.1\ES5\logstash-5.5.1\bin\mysql\station_parameter.txt"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      #statement_filepath => "config-mysql/test02.sql"
      statement => "select * from user u where u.birth > :sql_last_value"
      schedule => "* * * * *"
      #索引的类型
      type => "test02"
    }
}

filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}

output {
    elasticsearch {
    	# ES地址与端口
        hosts => "127.0.0.1:9200"
        # index名
        index => "test01"
        # 需要关联的数据库中有有一个id字段，对应索引的id号
        document_id => "%{id}"
    }
    stdout {
        codec => json_lines
    }
}
```



* 启动logstash

```shell
$ /usr/local/logstash-5.6.10/bin$ ./logstash -f config-mysql/mysql.conf
```





## 五、MySQL与ES数据同步

* 数据采集->MySQL->ES

### delete

* mysql中数据不删除，通过标志位逻辑删除，将状态同步到ES

### insert,update

*  logstash-input-jdbc 插件同步数据

[logstash-input-jdbc](https://blog.csdn.net/laoyang360/article/details/51747266)



## 分词器安装

* clone分词器到本地，ik分词器地址

```shell
https://github.com/medcl/elasticsearch-analysis-ik
```

* maven打包

```shell
$ mvn clean package -DskipTests
Building zip: E:\workspace\github\ik\elasticsearch-analysis-ik\target\releases\elasticsearch-analysis-ik-5.6.9.zip
```

* 上传zip包

```shell
/usr/local/elasticsearch-5.6.10/plugins/
```

* 另一种直接安装

```shell
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.5/elasticsearch-analysis-ik-5.6.5.zip
```









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

