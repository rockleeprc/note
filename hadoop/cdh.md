* #### 下载cloudera-manager

  - 下载与linux版本对应的，比如centos7使用[cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz](http://archive.cloudera.com/cm5/cm/5/cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz)

  ```html
  http://archive.cloudera.com/cm5/cm/5/
  ```

  #### 下载CDH安装包

  - 下载与linux版本对应的，比如centos7使用

    - [CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel](http://archive.cloudera.com/cdh5/parcels/5.16.1/CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel)
    - [CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1](http://archive.cloudera.com/cdh5/parcels/5.16.1/CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1)
    - [manifest.json](http://archive.cloudera.com/cdh5/parcels/5.16.1/manifest.json)

    ```html
    http://archive.cloudera.com/cdh5/parcels/latest/
    ```

* 所有节点免密

* 安装ntp

* 安装mysql，设置权限

  ```mysql
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
  flush privileges;
  ```

* 安装三方依赖包

  ```shell
  yum install -y chkconfig python bind-utils psmisc libxslt zlib sqlite cyrus-sasl-plain cyrus-sasl-gssapi fuse fuse-libs redhat-lsb 
  ```

* 在每节点上创建目录

  ```shell
  mkdir /opt/cloudera-manager
  ```

* 将Cloudera Manager Server 解压到该目录

  ```shell
  tar xvzf cloudera-manager*.tar.gz -C /opt/cloudera-manager
  ```

* 在个节点上创建启动Cloudera Manager Server的用户 cloudera-scm

  ```shell
  useradd --system --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
  ```

* 在`/opt/cloudera-manager/cm-5.4.3/etc/cloudera-scm-agent/config.ini`中配置server的主机名

  ```shell
  server_host=node1
  ```

* 将server节点的/opt/cloudera-manage目录分发到agent节点

* 在每个节点，拷贝mysql jar文件到目录 `/usr/share/java/` ，包名称要修改为mysql-connector-java.jar 

* 创建scm的专属用户

  ```mysql
  grant all on *.* to 'scm'@'%' identified by 'scm' with grant option;
  ```

* 将数据库连接写入配置文件，测试数据库连接

  ```shell
  cd /opt/cloudera-manager/cm-5.4.3/share/cmf/schema/
  ./scm_prepare_database.sh mysql scm -h node3 -uscm -pscm --scm-host node1 scm scm scm
  格式：数据库类型、数据库、数据库服务器、用户名、密码、cm server服务器
  ```

* 在每个节点准备Parcel目录，并 修改目录权限

  ```shell
  Server节点
  mkdir -p /opt/cloudera/parcel-repo
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
  Agent节点
  mkdir -p /opt/cloudera/parcels
  chown cloudera-scm:cloudera-scm /opt/cloudera/parcels
  
  ```

* 在server节点配置cdh本地源，将这三个文件放到server节点的/opt/cloudera/parcel-repo下
CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel
CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha(改文件名称)
manifest.json

* 启动 server、agent，Sever首次启动会自动创建表以及数据，不要立即关闭或重启，否则需要删除所有表及数据重新安装

  ```shell
  cd /opt/cloudera-manager/cm-5.4.3/etc/init.d/
  ./cloudera-scm-server start
  ./cloudera-scm-agent start
  ```

* 测试访问

  ```shell
  http://ManagerHost:7180 
  admin:admin
  ```

* 服务脚本

  ```shell
  ##amon
  create database amon      DEFAULT CHARACTER SET utf8;
  grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
  
  ##hive
  create database hive     DEFAULT CHARACTER SET utf8;
  grant all on hive.* TO 'hive'@'%' IDENTIFIED BY 'hive';
  
  ##oozie
  create database oozie     DEFAULT CHARACTER SET utf8;
  grant all on oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';
  
  ```

  