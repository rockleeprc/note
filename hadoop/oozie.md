## oozie模块/节点

### Workflow

- 顺序执行流程节点，支持fork（分支多节点）、join（合并多个节点）

### Coordinator

- 定时触发Workflow

### Bundle Job

- 绑定多个Coordinator

### Control Flow Nodes

- 定义流程开始或结束的位置，比如：start、end、kill、decision、fork、join

### Action Nodes

- 具体执行的动作，比如：拷贝、执行shell



- cdh配置oozie时区

  ```xml
  <!-- oozie-site.xml 的 Oozie Server 高级配置代码段 -->
  <property>
      <name>oozie.processing.timezone</name>
      <value>GMT+0800</value>  
  </property>
  ```



```shell
// 查看当前Oozie的share-lib共享库HDFS目录
oozie admin -oozie http://node5:11000/oozie -sharelibupdate
oozie admin -oozie http://temporaryProd1:11000/oozie -sharelibupdate
```



```shell
// extjs目录
cd /opt/cloudera/parcels/CDH/lib/oozie/libext
```



- 修改hue时区
- 修改oozie时区



### 安装

* 在/user/oozie/share/lib/lib_20191115175523/下创建spark2目录
* 将/user/oozie/share/lib/lib_20191115175523/spark下的jar包copy到spark2下

### 常用命令

```shell
// 更新共享库中的jar包
oozie admin -oozie http://node5:11000/oozie -sharelibupdate
oozie admin -oozie http://temporaryProd1:11000/oozie -sharelibupdate
// 查看当前Oozie的share-lib共享库HDFS目录
oozie admin -oozie http://node5:11000/oozie -shareliblist

oozie job -oozie http://hh:11000/oozie -config ./xxxx/job.properties -run，这里的xxxx就是各个文件夹名称
oozie job -oozie http://hh:11000/oozie -config ./xxxx/job.properties -submit （提交任务） 
oozie job -oozie http://hh:11000/oozie -start job_id (运行已经提交的任务) 
oozie job -oozie http://hh:11000/oozie -config ./xxxx/job.properties -run (submit和start命令合并) 
oozie job -oozie http://hh:11000/oozie -info job_id (获取任务信息) 
oozie job -oozie http://hh:11000/oozie -suspend job_id (暂停/挂起任务) oozie job -oozie http://hh:11000/oozie -rerun job_id (重新运行暂停的任务)
```

### hue

* 配置jar目录时，要使用`hdfs:`作为前缀

* 指定spark2使用的jar包

  ```shell
  oozie.action.sharelib.for.spark spark2
  ```

  

### jenkins

```shell
// 将打包后的文件传输到指定目录
curl -i -X PUT -T /var/jenkins_home/workspace/Deploy_huixiang_ods_etl/ods-etl/target/ods-etl-1.0-SNAPSHOT.jar --header "Content-Type: application/octet-stream" "http://172.17.146.245:14000/webhdfs/v1/Demo/ods-etl-1.0-SNAPSHOT.jar?op=CREATE&user.name=hdfs&buffersize=1000&data=true"
```









