

## 管理控制台的搭建
* 下载源码
	* https://github.com/alibaba/dubbo/tree/2.5.x
* 编译
	* 编译 mvn clean package
	* dubbo-admin/target生成dubbo-admin-2.5.10.war
* 修改配置
	* dubbo-admin-2.5.10/WEB-INF目录修改dubbo.properties
	* zookeeper地址、root.password、guest.password
* 容器部署
	* 拷贝dubbo-admin-2.5.10.war到apache-tomcat-7.0.10/webapps目录下并解压
	* 访问 http://127.0.0.1:8080/dubbo-admin-2.5.10/
	

	