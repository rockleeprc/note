
## 核心组件生命周期

### SqlSessionFactoryBuilder

通过XML或者Java编码获得资源，构建SqlSessionFactory（可以构建多个），作用就是一个构建器，生命周期存在于方法局部，生成SqlSessionFactory对象后丢弃

### SqlSessionFactory

创建SqlSession，生命周期存在于整个MyBatis应用中，每个数据库只对应一个SqlSessionFactory，避免Connection消耗

### SqlSession

相当于JDBC的Connection，线程不安全，生命周期在于请求数据库处理事物的过程中

### Mapper

一个interface没有实现类，MyBatis根据这个接口生成代理对象，代理对象根据接口全路径+方法名去匹配xml文件中的sql，生命周期在一个SqlSession事物内
