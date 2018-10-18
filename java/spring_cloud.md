## Eureka

### server pom

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

### server yml

* 单实例部署，register-with-eureka、fetch-registry为false
* 集群部署，分别修改server.port、eureka.instance.hostname

```yaml
server: 
  port: 7001

spring:
  application:
    name: eureka-clouster

eureka: 
  instance:
    hostname: peer1 #eureka服务端的实例名称
  client:
    #register-with-eureka: false     #false表示不向注册中心注册自己。
    #fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://peer2:7002/eureka/,http://peer3:7003/eureka/
```

### server

* 增加@EnableEurekaServer

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApp7001
{
    public static void main( String[] args ){
        SpringApplication.run(EurekaServerApp7001.class,args);
    }
}
```

### client provider pom

```xml
     <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

### client provider yml

* instance-id：client注册eureka后显示的实例名称
* prefer-ip-address：浏览器访问eureka server时显示eureka client的ip地址
* info：eureka client注册到server后，显示的一些信息

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://peer1:7001/eureka/,http://peer2:7002/eureka/,http://peer3:7003/eureka/
  instance:
    instance-id: microservicecloud-dept-8001
    prefer-ip-address: true     #访问路径可以显示IP地址
# eureka服务发现
info:
  app.name: microservicecloud-provider-dept
  company.name: www.foo.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

### client provider

```java

@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class DeptProviderApp9001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderApp9001.class, args);
    }
}
```

### client consumer pom

* 消费方需要负载均衡策略和eureka整合在一起，使用ribbon

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

### client consumer yml

* 消费方不注册自己

```yaml
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://peer1:7001/eureka/,http://peer2:7002/eureka/,http://peer3:7003/eureka/
```

### client consumer

* 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumerApp8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerApp8001.class, args);
    }
}
```

* 配置类

```java

@Configuration
public class ConfigBean {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule loadBalancderRule(){
        return new MyRandomRule();
    }
}
```

### 总结

* eureka server用的pom是`spring-cloud-starter-eureka-server`，client是`spring-cloud-starter-eureka`，eureka需要`spring-cloud-starter-config`
* 消费方在调用时需要负载均衡策略，使用`spring-cloud-starter-ribbon`，使用` @LoadBalanced`注解配置在RestTemplate上，使用RestTemplate时自动实现负载均衡策略
* 单实例和集群配置`register-with-eureka`、`fetch-registry`两个参数配置不一样
* @ComponentScan描到@Configuration配置的负载均衡策略，对所有微服务有效，针对某一个微服务时@Configuration不能被@ComponentScan扫描到