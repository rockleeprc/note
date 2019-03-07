## 安装

### 集群

* 配置文件目录$spark_home/conf 

* `slaves`中加入savle主机名

  ```shell
  node1
  node2
  node3
  node4
  ```

* `spark-env.sh`配置master主机名、端口、历史服务器启动参数

  ```shell
  # master主机名
  SPARK_MASTER_HOST=node1
  # master端口
  SPARK_MASTER_PORT=7077
  # 环境变量
  export JAVA_HOME=/usr/local/jdk
  export SCALA_HOME=/usr/local/scala
  export HADOOP_HOME=/usr/local/hadoop
  # 历史服务器启动参数配置
  export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://node1:9000/directory"
  ```

* `spark-default.conf`配置日志

  ```shell
  spark.eventLog.enabled  true
  spark.eventLog.dir       hdfs://node1:9000/directory
  spark.eventLog.compress true
  ```

* 将配置分发到其它节点

* 启动spark $spark_home/sbin/start-all.sh

* 启动spark历史服务器 $spark_home/sbin/start-history-server.sh

* 权限问题在hdfs-site.xml中配置关闭权限

  ```shell
  <property>
      <name>dfs.permissions</name>
      <value>false</value>
  </property>  
  ```

* 提交任务测试集群

  ```spark
  $spark_home/bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://master01:7077 \
  --executor-memory 1G \
  --total-executor-cores 2 \
  /$spark_home/examples/jars/spark-examples_2.11-2.1.1.jar \
  100
  
  ```

## RDD

* 分区

  ```shell
  # 不设置分区数
  > val part=sc.textFile("")
  # 设置分区数为6
  > val part=sc.textFile("",6)
  > part.partitions.size	
  ```

* 依赖关系

  ```shell
  > val rdd = sc.textFile("")
  > val wordmap = rdd.flotMap(rdd.split(" ")).map(x => (x,1))
  > println(wordmap)
  # 窄依赖
  > wordmap.dependencies.foreach{
      dep => {
      	println(dep.getClass)
      	println(dep.rdd)
      	println(dep.rdd.partitions)
      	println(dep.rdd.partitions.length)
          }
  }
  > val wordreduce=wordmap.reduceByKey(_+_)
  > println(wordreduce)
  # 宽依赖
  > wordreduce.dependencies.foreach(dep => {
      dep => {
      	println(dep.getClass)
      	println(dep.rdd)
      	println(dep.rdd.partitions)
      	println(dep.rdd.partitions.length)
         	}
  })
  ```

* mapPartitions()，以partition为计算单位，对迭代器进行操作

  ```shell
  > val a = sc.parallelize(1 to 9,3)
  > a.mapPartitions(iter=>{
      var res = List()
      var pre = iter.next
      # 迭代器以分区为单位进行计算
      while(iter.hasNext){
          val cur = iter.next
          res ::= (pre,cur)
          pre = cur
      }
      res.iterator
  }).collect
  ```

* 分区函数，(K,V)类型有HashPartitioner、RangerPartitoiner，非(K,V)类型Partitioner=None

  ```shell
  > val part = sc.textFile("")
  # 分区函数=None
  > part.partitioner
  > val rdd = part.map(x=>(x,x)).groupByKey(new org.apache.spark.HashPartitioner(4))
  # 查看partitioner
  > rdd.partitioner
  ```

* 创建操作

  ```shell
  > val rdd = sc.parallelize(1 to 10)
  # 设置4个分区
  > val rdd = sc.parallelize(1 to 10,4) 
  > val rdd = sc.makeRDD(1 to 10) 
  > rdd.partitions.size
  # 查看分区
  > rdd.preferredLocations(rdd.partitions(0)) 
  ```

### 创建操作

* 存储

  ```shell
  # 读取本地文件系统时，每台机器都要有该文件
  > val rdd = sc.textFile("hdfs://")
  > rdd.count
  # 可以指定partiton的数量，默认是hdsf block分片数，自定义时不能少于block分片数
  > val rdd = sc.textFile("hdfs://",4)
  ```

### 转换操作

* map：对rdd中每个元素执行一个函数，产生新的rdd

* distinct：去除rdd中重复的元素，返回不重复的rdd

* flatMap：将rdd中每一个元素生成一个或多个元素来构建新的rdd

  ```shell
  > val data = sc.textFile("")
  # 每行数据一个Array
  > sc.map(line => line.split("")).collect
  # 产生一个Array
  > sc.flatMap(_.split("")).collect
  > sc.flatMap(_.split()).distinct.collect
  ```

* coalesce：使用hashPartitoner进行重分区，默认不使用shuffle

* repartiton：和coalesce函数一样，使用shuffle

  ```shell
  > val data = sc.textFile("")
  # 查看分区数
  > data.partitions.size
  # 重分区的数目小于原分区数目，可以
  > val rdd = data.coalesce(1)
  # 重分区的数目大于原分区数目，shuffle必须为true，否则分区数不会改变
  > val rdd = data.coalesce(1，true)
  ```

* randomSplit：根据weights权重将一个RDD分割为多个RDD

* glom：将每一个分区中所有类型T的数据转变为Array[T]的嵌套数组

  ```shell
  # 生成一个集合，10个分区
  > val rdd = sc.makeRDD(1 to 10,10)
  # 返回的是一个RDD数组
  > val split = rdd.randomSplit(Array(1.0,2.0,3.0,4.0))
  # 查看RDD数组长度
  > split.size
  > split(0).collect
  
  > val rdd sc.makeRDD(1 to 10,3)
  # 三个分区，生成Array(Array(),Array(),Array())
  > rdd.glom().collect
  ```

* union：返回两个RDD并集，不去重

* intersection：返回两个RDD交集，去重

* subtract：返回在RDD中出现，且不在otherRDD中出现的元素

  ```shell
  > val rdd1 = sc.makeRDD(1 to 2,1)
  > val rdd2 = sc.makeRDD(2 to 3,1)
  # 并集，不去重
  > rdd1.union(rdd2).collect 
  # 交集，去重
  > rdd1.intersection(rdd2).collect
  > rdd1.subtract(rdd2).coolect
  ```

* mapPartitions：使用迭代器处理RDD中每一个分区的数据

* mapPartitionsWithIndex：使用迭代器处理RDD中每一个分区的数据，函数中带分区标识

  ```shell
  > val rdd = sc.makeRDD(1 to 5,2)
  # 对每个分区内数据进行累加
  > val res = rdd.mapPartitions(iter=>{
      val resutl = List[Int]()
      var i = 0
      while(iter.hasNext){
          i+=iter.next
      }
      result.::(i).iterator
  }).collect
  > val rdd: RDD[Int] = sc.makeRDD(1 to 10, 3)
      rdd.mapPartitionsWithIndex((idx, iter) => {
        while (iter.hasNext) {
          println(s"$idx-" + iter.next())
        }
        iter
      }).collect()
  ```

* partitionBy：根据partitioner函数将原RDD重新分区，生成新的ShuffleRDD，使用groupByKey

* mapValues：针对[K,V]中的V进行map操作

* flatMapValues：针对[K,V]中的V进行flatMap操作

  ```shell
  > val rdd = sc.makdeRDD(Array)
  > rdd.mapValues(v=>{
        println(v)
        }).collect()
  ```

* combineByKey：将RDD[K,V]转换为RDD[K,C]，V、C类型可以相同也可以不同

  ```shell
  val rdd: RDD[(String, String)] = sc.makeRDD(Array(("A","a"),("B","b"),("B","bb"),("B","bbb"),("B","bbbb"),("C","c"),("D","d")),2)
      val newRdd: Array[(String, String)] = rdd.combineByKey(
        (v:String)=>v+"#",
        (c:String,v:String)=>c+"_"+v,
        (c1:String,c2:String)=>c1+"*"+c2).collect
      newRdd.foreach(x=>{
        println(x._2+"-"+x._1)
      })
  ```

* foldByKey：

### 行动

* first：返回RDD中的第一个元素，不排序
* count：返回RDD中元素个数
* reduce：对RDD中的元素进行二元运算
* collect：将RDD转换为数组



