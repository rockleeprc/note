## 继承AbstractRoutingDataSource
* 继承`org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource`类，实现`org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource#determineCurrentLookupKey`
* 实现`org.springframework.beans.factory.InitializingBean#afterPropertiesSet`，初始化`targetDataSources`、`defaultTargetDataSource`变量，现调用父类方法
* 调用`org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource#getConnection()`通过`determineCurrentLookupKey`获取datasource

## 数据源切换
* MyBatis Interceptor，适合读写分离
* AOP，适合多业务库
    * 声明自定义注解
    * 在Service层添加注解，AOP通过拦截注解设置数据源（前置、环绕）
* 集成多SqlSessionFactory，针对多DataSource设置不同的食物管理器

## 事务处理
* 编程事物，手动处理
* 嵌套事物，注解处理
    ```java
    @EnableAspectJAutoProxy(exposeProxy=true)
    AopContext.currentProxy(); //获取当前方法的代理对象 
    // 不建议在Service自己注入自己，springboot2.6默认禁止
    ```