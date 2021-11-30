# 源码解析

## 自动装配机制

* pom中引入的`spring-cloud-starter-alibaba-sentinel`包下`spring.factories`文件包含所有与sentinel相关的自动装配类

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.alibaba.cloud.sentinel.SentinelWebAutoConfiguration,\
com.alibaba.cloud.sentinel.SentinelWebFluxAutoConfiguration,\
com.alibaba.cloud.sentinel.endpoint.SentinelEndpointAutoConfiguration,\
com.alibaba.cloud.sentinel.custom.SentinelAutoConfiguration,\
com.alibaba.cloud.sentinel.feign.SentinelFeignAutoConfiguration

org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker=\
com.alibaba.cloud.sentinel.custom.SentinelCircuitBreakerConfiguration
```

## @SentinelResource注解解析机制

* 在`com.alibaba.cloud.sentinel.custom.SentinelAutoConfiguration`配置类中注入`com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect`，包含处理`@SentinelResource`注解相关逻辑
* 使用`@Around`拦截`@SentinelResource`注解，调用`SphU.entry()`设置流控、降级等策略

## ProcessorSlotChain初始化

* 通过`com.alibaba.csp.sentinel.slots.DefaultSlotChainBuilder`初始化`ProcessorSlotChain`实例

* 通过JDK SPI机制获取`sentinel-core-1.8.1.jar`包下所有`com.alibaba.csp.sentinel.slotchain.ProcessorSlot`实现类

  ```properties
  # Sentinel default ProcessorSlots
  com.alibaba.csp.sentinel.slots.nodeselector.NodeSelectorSlot
  com.alibaba.csp.sentinel.slots.clusterbuilder.ClusterBuilderSlot
  com.alibaba.csp.sentinel.slots.logger.LogSlot
  com.alibaba.csp.sentinel.slots.statistic.StatisticSlot
  com.alibaba.csp.sentinel.slots.block.authority.AuthoritySlot
  com.alibaba.csp.sentinel.slots.system.SystemSlot
  com.alibaba.csp.sentinel.slots.block.flow.FlowSlot
  com.alibaba.csp.sentinel.slots.block.degrade.DegradeSlot
  ```

* 所有的`com.alibaba.csp.sentinel.slotchain.ProcessorSlot`添加到`ProcessorSlotChain`后的结构，类似尾插法

  ![image-20211127141826360](/Users/admin/WorkSpace/github/prc/note/java/sentinel.assets/image-20211127141826360.png)

* 所有solt在责任链中的顺序通过`com.alibaba.csp.sentinel.spi.Spi`确认，顺序从小到大

* 责任链在`ProcessorSlotChain`中的调用逻辑

  ```java
  // com.alibaba.csp.sentinel.slotchain.DefaultProcessorSlotChain#entry
  @Override
  public void entry(Context context, ResourceWrapper resourceWrapper, Object t, int count, boolean prioritized, Object... args)
    throws Throwable {
    // 从first开始向下调用
    first.transformEntry(context, resourceWrapper, t, count, prioritized, args);
  }
  
  AbstractLinkedProcessorSlot<?> first = new AbstractLinkedProcessorSlot<Object>() {
  
    @Override
    public void entry(Context context, ResourceWrapper resourceWrapper, Object t, int count, boolean prioritized, Object... args)
      throws Throwable {
      // 父类处理责任链调用逻辑
      super.fireEntry(context, resourceWrapper, t, count, prioritized, args);
    }
  
    @Override
    public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
      super.fireExit(context, resourceWrapper, count, args);
    }
  
  };
  
  // com.alibaba.csp.sentinel.slotchain.AbstractLinkedProcessorSlot
  @Override
  public void fireEntry(Context context, ResourceWrapper resourceWrapper, Object obj, int count, boolean prioritized, Object... args)
    throws Throwable {
    if (next != null) {
      // 责任链调用逻辑
      next.transformEntry(context, resourceWrapper, obj, count, prioritized, args);
    }
  }
  ```

  ## Slot执行逻辑及异常处理

* 以`com.alibaba.csp.sentinel.slots.block.flow.FlowSlot#entry`执行逻辑为例

  ```java
  // com.alibaba.csp.sentinel.slots.block.flow.FlowSlot#entry
  @Override
  public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, boolean prioritized, Object... args) throws Throwable {
    // 流控规则检查
    checkFlow(resourceWrapper, context, node, count, prioritized);
    // 调用下一个slot
    fireEntry(context, resourceWrapper, node, count, prioritized, args);
  }
  
  // com.alibaba.csp.sentinel.slots.block.flow.FlowRuleChecker#checkFlow
  public void checkFlow(Function<String, Collection<FlowRule>> ruleProvider, ResourceWrapper resource, Context context, DefaultNode node, int count, boolean prioritized) throws BlockException {
    if (ruleProvider == null || resource == null) {
      return;
    }
    // 从内存中获取流控配置规则
    Collection<FlowRule> rules = ruleProvider.apply(resource.getName());
    if (rules != null) {
      for (FlowRule rule : rules) {
        // 是否能通过校验
        if (!canPassCheck(rule, context, node, count, prioritized)) {
          // 不能通过直接抛异常，责任链不会向下执行
          throw new FlowException(rule.getLimitApp(), rule);
        }
      }
    }
  }
  ```

* solt异常处理逻辑

  * 在`com.alibaba.csp.sentinel.slots.block.flow.FlowSlot`抛出异常后，`com.alibaba.csp.sentinel.slots.statistic.StatisticSlot#entry`中在try...catch获取到异常，设置一些参数后继续向上层throw

  * 在`com.alibaba.csp.sentinel.slots.statistic.StatisticSlot#entry`抛出异常后，`com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect#invokeResourceWithSentinel`中在try...catch获取到异常后，通过`@SentinelResource`配置的降级方法进行后续处理，具体逻辑在`com.alibaba.csp.sentinel.annotation.aspectj.AbstractSentinelAspectSupport#handleBlockException`

  * 伪代码逻辑

    ```java
    // AOP拦截@SentinelResource
    try{
      // ...
      // slot责任链调用逻辑，Slot -> StatisticSlot -> FlowSlot -> Slot
      // StatisticSlot调用
      try{
        // FlowSlot调用，抛出FlowException
      }catch(BlockException e){
        // 设置参数
        throw e
      }
    }catch(BlockException ex){
      // 解析@SentinelResource获取降级方法，执行降级策略
    }
    ```

    ## 扩展点

    ### 第一次请求初始化

    * `com.alibaba.csp.sentinel.init.InitFunc`用于在请求第一次到达资源时做初始化操作
    
    * 在`com.alibaba.csp.sentinel.init.InitExecutor#doInit`中通过JDK SPI机制加载`com.alibaba.csp.sentinel.init.InitFunc`接口实现类，通过`com.alibaba.csp.sentinel.init.InitOrder`注解排序
    
      ```java
      public class Env {
          public static final Sph sph = new CtSph();
          static {
              // 通过SPI机制加载InitFunc实现类
              InitExecutor.doInit();
          }
      }
      ```
    
    
    
    
    
    
    
    
    
    
    