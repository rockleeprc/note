



```shell
curl -i -X PUT -T /var/jenkins_home/workspace/Deploy_huixiang_ods_etl/ods-etl/target/ods-etl-1.0-SNAPSHOT.jar
 --header "Content-Type: application/octet-stream" "http://172.17.146.245:14000/webhdfs/v1/Demo/ods-etl-1.0-SNAPSHOT.jar?op=CREATE&user.name=hdfs&buffersize=1000&data=true"
```

