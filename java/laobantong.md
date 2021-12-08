# laobantong-service升级改造

## 升级版本

* `spring-boot-dependencies:2.3.2.RELEASE`
* `spring-cloud-alibaba-dependencies:2.2.6.RELEASE`
* `grpc-spring-boot-starter:3.0.1-SNAPSHOT`
* `monitor-spring-boot-starter:1.0.0-SNAPSHOT`

## 改造点

* 项目启动通过`org.springframework.boot.SpringApplication`
* `spring.profiles.active`没有通过spring启动参数，使用JVM启动参数
* 通过`javax.annotation.PostConstruct`声明周期加载配置
* 通过`io.grpc.ServerInterceptor`暴露`grpc_request_total`、`grpc_response_time_milliseconds`监控指标
* 所有日志通过slf4j适配logback
* 通过`com.hualala.report.dao.impl.sql.run.ReportDaoExecutor`计算sql执行时间，写入log日志

# laobantong-api升级改造

## 升级版本

* `spring-boot-dependencies:2.3.2.RELEASE`
* `spring-cloud-alibaba-dependencies:2.2.6.RELEASE`

## 改造点

### 缓存策略

* `com.hualala.laobantong.annotation.RedisCacheAside`

  ![image-20211130161055659](/Users/admin/WorkSpace/github/prc/note/java/laobantong.assets/image-20211130161055659.png)

* `com.hualala.laobantong.annotation.RedisCacheAlwaysWrite`

  ![image-20211130161111024](/Users/admin/WorkSpace/github/prc/note/java/laobantong.assets/image-20211130161111024.png)

### 缓存key设计

* 采用hash结构保存数据
* 使用controller全类名作为key，使用请求参数murmur128后作为field
  * contoller作为key目的是减小key的数量
  * murmur128作为field，long类型、定长，减小内存占用、减小内存碎片
    * 经过测试5000万个murmur128只占381.46973MB
    * 经过测试1亿个murmur128只占762.93945MB
* key序列化使用`org.springframework.data.redis.serializer.StringRedisSerializer`，value序列化使用`com.hualala.laobantong.config.RedisLettuceConfig.ProtoStuffRedisSerializer`

# sentinel

## 持久化策略

* 流控、降级持久化到zookeeper，热点、授权未持久化

## 配置发布策略

* 支持服务名称、服务名称+ip、服务名称+ip+port

* 最终选择基于服务名称发布配置，k8s每次部署无法绑定特定ip

  ![image-20211130164354256](/Users/admin/WorkSpace/github/prc/note/java/laobantong.assets/image-20211130164354256.png)
