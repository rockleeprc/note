

# 环境准备

## pom

```xml
<spring-boot-version>2.3.2.RELEASE</spring-boot-version>
<spring-cloud-version>Hoxton.SR9</spring-cloud-version>
<spring-cloud-alibaba-version>2.2.6.RELEASE</spring-cloud-alibaba-version>
<camunda-version>7.16.0</camunda-version>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring-boot-version}</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>${spring-cloud-alibaba-version}</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
</dependencies>
<!-- camunda核型依赖 -->
<dependency>
  <groupId>org.camunda.bpm.springboot</groupId>
  <artifactId>camunda-bpm-spring-boot-starter</artifactId>
  <version>${camunda-version}</version>
</dependency>
<!-- web管理界面 -->
<dependency>
  <groupId>org.camunda.bpm.springboot</groupId>
  <artifactId>camunda-bpm-spring-boot-starter-webapp</artifactId>
  <version>${camunda-version}</version>
</dependency>
<!-- reset api -->
<dependency>
  <groupId>org.camunda.bpm.springboot</groupId>
  <artifactId>camunda-bpm-spring-boot-starter-rest</artifactId>
  <version>${camunda-version}</version>
</dependency>
```

## processes.xml

![image-20211222201130170](/Users/admin/WorkSpace/github/prc/note/java/workflow.assets/image-20211222201130170.png)

## 配置文件

```yaml
spring:
  application:
    name: camunda-exmaple
  web:
    resources:
      static-locations: NULL
  datasource:
    url: jdbc:mysql://172.20.44.11:3306/camunda?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=utf-8&useSSL=false&nullCatalogMeansCurrent=true # 测试环境
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456

mybatis:
  configuration:
    map-underscore-to-camel-case: true
#    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

camunda:
  bpm:
    admin-user:
      id: admin
      password: admin # 管理页面 用户名
      firstName: admin
    filter:
      create: All tasks
    auto-deployment-enabled: false # 是否自动部署

# 打印sql
logging:
  level:
    org.camunda.bpm.engine: INFO

```

## 核心api

### RepositoryService

* 流程部署

* 查询流程定义

* 删除流程定义

### RuntimeService

* 启动流程

* 终止流程

### TaskService

* 查询任务（Assignee、CandidateUser）

* 任务认领、任务退还、任务转发

* 任务完成

### HistoryService

* 获取任务历史

### IdentityService

* 获取用户
* 获取租户

# 流程工具

## camunda-modeler获取

```markdown 
# 下载
https://camunda.com/download/modeler/
# brew
brew install camunda-modeler
```

## 流程节点

## 案例1

![image-20211222105332820](/Users/admin/WorkSpace/github/prc/note/java/workflow.assets/image-20211222105332820.png)

## 案例2

![image-20211222201950590](/Users/admin/WorkSpace/github/prc/note/java/workflow.assets/image-20211222201950590.png)

# 核型表结构

```markdown
# GE 表示 general，通用数据，用于不同场景下
ACT_GE_* 
# RU 表示 runtime，这些运行时的表，包含流程实例、任务、变量、异步任务等运行中的数据。只在流程实例执行过程中保存这些数据，在流程结束时就会删除这些记录，这样来尽量使运行时表可以一直很小速度很快
ACT_RU_*
# HI 表示 history，这些表包含历史数据，比如历史流程实例、遍历、任务等等
ACT_HI_* 
# RE 表示 repository，这个前缀的表包含了流程定义和流程静态资源(图片，规则，等等)
ACT_RE_* 
```

## 通用表

```markdown
# 资源（二进制数据）表
act_ge_bytearray 
# 流程引擎级别的数据表
act_ge_property
```

## 仓库表

```markdown
# 流程部署表
act_re_deployment 
# 流程定义表
act_re_procdef 
```

## 运行时表

```markdown

# 运行时流程实例(执行流)表
act_ru_execution 
# 流程与身份关系表(执行流)(候选人代办)
act_ru_identitylink (act_hi_identitylink)
# 运行时流程任务(节点)表(执行流)
act_ru_task 
# 运行时流程参数(变量)表(执行流)
act_ru_variable(act_hi_varinst)
# 运行时定时任务
act_ru_job
```

## 历史表

```markdown
# 流程实例（历史）表
act_hi_procinst 
# 流程明细（历史）表
act_hi_detail 
# 历史任务实例表
act_hi_taskinst 
# 历史节点（行为）表
act_hi_actinst 
# 历史附件表
act_hi_attachment 
# 历史意见表
act_hi_comment 
# 历史详情表
act_hi_detail
```

## 用户表

```markdown
# 用户组信息表
act_id_group
# 用户信息表
act_id_user
# 用户于组关系表，中间表
act_id_membership
```





