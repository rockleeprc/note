
## Spring+Mybatis
* 保留mybatis自己的配置文件，但只配置settings
* mappers通过org.mybatis.spring.SqlSessionFactoryBean.mapperLocations扫描
* typeAliases通过org.mybatis.spring.SqlSessionFactoryBean.typeAliasesPackage

		<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<!-- 实例化sqlSessionFactory时需要使用上述配置好的数据源以及SQL映射文件 -->
			<property name="dataSource" ref="dataSource" />
			<!-- mybatis 配置文件 -->
			<property name="configLocation" value="classpath:mybatis/mybatis.xml" />
			<!-- 自动扫描exam/dao/目录下的所有SQL映射的xml文件, 省掉mybatis.xml里的手工配置value="classpath:exam/dao/*.xml" 
				指的是classpath(类路径)下exam.dao包中的所有xml文件 UserMapper.xml位于exam.dao包下，这样UserMapper.xml就可以被自动扫描 -->
			<property name="mapperLocations" value="classpath:exam/dao/*Mapper.xml" />
			<!-- 将包下的类名映射为别名 -->
			<property name="typeAliasesPackage" value="exam.pojo"/>
		</bean>

* 扫描Mapper使用MapperScannerConfigurer配置
* 生成代理使用MapperFactoryBean
 
	 	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
			<!-- 扫描exam.dao这个包以及它的子包下的所有映射接口类 -->
			<property name="basePackage" value="exam.dao" />
			<!-- <property name="sqlSessionFactory" ref="sqlSessionFactory"></property> 
				java.sql.SQLException: unkow jdbc driver : ${jdbc_url} ref时会马上实例化sqlSessionFactory,但properties文件还没有加载 -->
			<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
			<!-- 被标注Repository才扫描 -->
			<property name="annotationClass" value="org.springframework.stereotype.Repository"/>
		</bean>