## 部署地址

```markdown
# qualits
192.168.101.12
# qualits
/home/hadoop/qualitis/qualitis-0.7.0

# linkis
192.168.101.153
# linkis 部署位置
/disk6/linkis-0.9.3/linkis-ujes-shell-entrance
/disk6/linkis-0.9.3/linkis-bml
/disk6/linkis-0.9.3/linkis-ujes-hive-entrance
/disk6/linkis-0.9.3/linkis-gateway
/disk6/linkis-0.9.3/linkis-resourcemanager
/disk6/linkis-0.9.3/linkis-publicservice
/disk6/linkis-0.9.3/linkis-ujes-spark-entrance
/disk6/linkis-0.9.3/linkis-ujes-python-entrance
/disk6/linkis-0.9.3/linkis-ujes-spark-enginemanager

# 晚上暂停，早上启动
start-spark-entrance.sh
start-spark-enginemanager.sh

# nginx 
192.168.101.93
```



## 任务无法提交问题排查

### qualits项目

```markdown
192.168.101.12

# pwd
/home/hadoop/qualitis/qualitis-0.7.0

# error log
2021-11-30 00:08:08,037 ERROR [schedulerFactoryBean_Worker-1] quartz.QualitisJob execute: job执行失败, caused by:
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "http://192.168.101.93:19999/api/rest_j/v1/entrance/execute": 400 Bad Request; nested exception is com.webank.wedatasphere.qualitis.exception.HttpRestTemplateException: 400 Bad Request
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:732) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:680) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        at org.springframework.web.client.RestTemplate.postForObject(RestTemplate.java:435) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        at com.webank.wedatasphere.qualitis.client.LinkisJobSubmitter.submitJob(LinkisJobSubmitter.java:98) ~[core_scheduler-0.7.0.jar:?]
        at com.webank.wedatasphere.qualitis.submitter.impl.ExecutionManagerImpl.submitApplication(ExecutionManagerImpl.java:132) ~[core_scheduler-0.7.0.jar:?]
        at com.webank.wedatasphere.qualitis.quartz.QualitisJob.execute(QualitisJob.java:91) [core_scheduler-0.7.0.jar:?]
        at org.quartz.core.JobRunShell.run(JobRunShell.java:202) [quartz-2.2.1.jar:?]
        at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:573) [quartz-2.2.1.jar:?]
Caused by: com.webank.wedatasphere.qualitis.exception.HttpRestTemplateException: 400 Bad Request
        at com.webank.wedatasphere.qualitis.handler.RestErrorHandler.handleError(RestErrorHandler.java:44) ~[web_app-0.7.0.jar:?]
        at org.springframework.web.client.ResponseErrorHandler.handleError(ResponseErrorHandler.java:63) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        at org.springframework.web.client.RestTemplate.handleResponse(RestTemplate.java:766) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:724) ~[spring-web-5.0.8.RELEASE.jar:5.0.8.RELEASE]
        ... 7 more
```

### nginx

```markdown
192.168.101.93

# pwd
/etc/nginx/conf.d/gauss.conf
# nginx 配置
location /api {
  proxy_pass http://upstream_api_sso; #后端Linkis的地址
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header x_real_ipP $remote_addr;
  proxy_set_header remote_addr $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_http_version 1.1;
  proxy_connect_timeout 4s;
  proxy_read_timeout 600s;
  proxy_send_timeout 12s;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection upgrade;
}
upstream upstream_api_sso {
	server  192.168.101.153:9001;
}

# error log
192.168.101.12 - - [02/Dec/2021:05:00:01 +0800] "POST /api/rest_j/v1/entrance/execute HTTP/1.1" 400 447 "-" "Apache-HttpClient/4.5.4 (Java/1.8.0_171)" "-"
```

### linkis

```markdown
192.168.101.153

# pwd
/disk6/linkis-0.9.3/linkis-gateway/
# pwd
/disk6/linkis-0.9.3/eureka
http://192.168.101.153:20303/eureka/
```

* error log，error太多

  ![image-20211202151309174](/Users/admin/Documents/work/qualitis.assets/image-20211202151309174.png)

  ![image-20211202151822652](/Users/admin/Documents/work/qualitis.assets/image-20211202151822652.png)

  ![image-20211202174029226](/Users/admin/Documents/work/qualitis.assets/image-20211202174029226.png)

  ![image-20211202180614845](/Users/admin/Documents/work/qualitis.assets/image-20211202180614845.png)

# 修改

```markdown
# web/app/src/main/resources/quartz.properties
org.quartz.dataSource.qzDS.URL = jdbc:mysql://172.20.44.11:3306/qualitis_new?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=utf-8&useSSL=false

# web/app/src/main/resources/application-dev.yml
address: 172.20.119.175:2181,172.20.47.150:2181,172.20.101.187:2181
server-url-prefix: http://127.0.0.1:18443/cas

core/project/src/main/java/com/webank/wedatasphere/qualitis/rule/entity/RuleAlert.java
```

# 表结构

```markdown
# /qualitis/api/v1/projector/rule/map
qualitis_rule_datasource
qualitis_application_task_rule_simple
qualitis_application_task_datasource

# post /api/v1/projector/rule
# 质量规则
qualitis_rule

# put /api/v1/projector/rule
qualitis_rule 
qualitis_rule_variable
qualitis_rule_alarm_config
qualitis_template_output_meta
qualitis_rule_datasource
qualitis_rule_alert
qualitis_rule_timing
```

# gradle

```markdown
# 定义项目lib与version
gradle/dependencies.gradle
```

