## 监控与管理
依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
### 原生端点
使用原生站点要配置，不然无法访问
```
management.security.enabled=false
```
#### 应用配置
/autoconfig：获取应用的自动化配置报告
/beans：获取上下文自动创建的bean
/configprops：应用中配置的属性报告
/env：环境属性
/mappings:springMVC所有的Controller映射关系

#### 度量指示
/metrics：返回应用的内存、线程、垃圾回收等信息
  /metrics/mem.free：通过key访问特定的信息
/health：各类健康指标
/dump：暴露程序中的线程信息
/trace：返回基本的http跟踪信息，保留最近的100条

#### 操作控制
/shutdown：关闭应用，需要如下配置
```
endpoints.shutdown.enabled=true
```

## 服务治理
服务治理围绕着服务注册和服务发现，对服务实施自动化管理
Eureka服务器端是一个注册中心，Eureka客户端（注解和配置）提供服务的注册和发现，服务器端和客户端都有java编写

## 保护模式
心跳失败比例在 15 分钟之内低于 85%
自我保护模式被激活的条件是：在 1 分钟后，Renews (last min) < Renews threshold
Renews threshold：(int)(服务个数 * 2 * 0.85)
Renews (last min)：服务个数*2
# 关闭自我保护
eureka.server.enable-self-preservation
# 默认0.85
eureka.server.renewal-percent-threshold


