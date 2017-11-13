

## 安装

* 环境变量：
	* M2_HOME
	* MAVEN_HOME

添加环境变量到PATH

	C:\Users\Administrator>mvn -version
	Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-
	Maven home: D:\Program Files\apache-maven\bin\..
	Java version: 1.8.0_112, vendor: Oracle Corporation
	Java home: D:\Java\jdk1.8.0_112\jre
	Default locale: zh_CN, platform encoding: GBK
	OS name: "windows 7", version: "6.1", arch: "amd64", family: "dos"

## mvn命令

### create/generate
	# 3.0.5前
	$ mvn archetype:create -DgroupId=com.tadu.orignal -DartifactId=project -DarchetypeArtifactId=maven-archetype-webapp
	# 3.0.5后
	$ mvn archetype:generate -DgroupId=com.tadu.orignal -DartifactId=project -DarchetypeArtifactId=maven-archetype-webapp
	[INFO] Using property: groupId = com.tadu.orignal
	[INFO] Using property: artifactId = project
	Define value for property 'version' 1.0-SNAPSHOT: :
	[INFO] Using property: package = com.tadu.orignal
	Confirm properties configuration:
	groupId: com.tadu.orignal
	artifactId: project
	version: 1.0-SNAPSHOT
	package: com.tadu.orignal
	 Y: : y
	[INFO] ----------------------------------------------------------------------------
	[INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-webapp:1.0
	[INFO] ----------------------------------------------------------------------------
	[INFO] Parameter: basedir, Value: E:\maven\pro
	[INFO] Parameter: package, Value: com.tadu.orignal
	[INFO] Parameter: groupId, Value: com.tadu.orignal
	[INFO] Parameter: artifactId, Value: project
	[INFO] Parameter: packageName, Value: com.tadu.orignal
	[INFO] Parameter: version, Value: 1.0-SNAPSHOT
	[INFO] project created from Old (1.x) Archetype in dir: E:\maven\pro\project
	[INFO] BUILD SUCCESS

###  compile
	# 生成target目录
	$ mvn compile

### test
	# 运行测试代码
	$ mvn test
	# 指定运行的测试类
	$ mvn test -Dtest=${page.class}

### clean
	# 删除target文件夹
	$ mvn clean
	[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ project ---
	[INFO] Deleting E:\maven\pro\project\target

### package
	$ mvn package
	[INFO] --- maven-war-plugin:2.2:war (default-war) @ project ---
	[INFO] Packaging webapp
	[INFO] Assembling webapp [project] in [E:\maven\pro\project\target\project]
	[INFO] Processing war project
	[INFO] Copying webapp resources [E:\maven\pro\project\src\main\webapp]
	[INFO] Webapp assembled in [26 msecs]
	[INFO] Building war: E:\maven\pro\project\target\project.war

### install
	# 将项目发布到本地仓库
	$ mvn install
	[INFO] Installing E:\maven\pro\project\target\project.war to E:\MavenRepository\com\tadu\orignal\project\1.0-SNAPSHOT\project-1.0-SNAPSHOT.war
	[INFO] Installing E:\maven\pro\project\pom.xml to E:\MavenRepository\com\tadu\orignal\project\1.0-SNAPSHOT\project-1.0-SNAPSHOT.pom

### tomcat:run
使用maven内嵌的tomcat


## scope生命周期

* compile：编译范围，默认scope，在classpath中存在，存活整个生命周期
* provided：已提供范围，比如容器提供Servlet API，不参与打包码，只参与编译测试
* runtime：运行时范围，编译不需要，接口与实现分离，不参与编译，参与打包
* test：测试范围，单元测试环境需要，只在测试时使用
* system：系统范围，自定义构件，指定systemPath，引入外部系统的jar包

## 多模块

父项目packaging:pom，在其中引入子项目 ，以后只对父项目维护
  	
	<packaging>pom</packaging>

	<modules>
		<module>../ordermanager</module>
		<module>../usermanager</module>
	</modules>

## 继承

在子项目中配置父项目，子项目中的<groupId>、<artifacctId>可以删除
	
	<modelVersion>4.0.0</modelVersion>
	<artifactId>usermanager</artifactId>
	<packaging>war</packaging>
	<name>usermanager Maven Webapp</name>
	<url>http://maven.apache.org</url>

	<parent>
		<groupId>tadu</groupId>
		<artifactId>parent</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

## 多模块+继承


## 添加jar包到maven本地仓库
	
	# 添加kaptcha-2.3.jar到本地maven仓库
	D:\>mvn install:install-file -Dfile=c:\kaptcha-2.3.jar -DgroupId=com.google.code 
	-DartifactId=kaptcha -Dversion=2.3 -Dpackaging=jar
	[INFO] Scanning for projects...
	[INFO] Searching repository for plugin with prefix: 'install'.
	[INFO] ------------------------------------------------------------------------
	[INFO] Building Maven Default Project
	[INFO]    task-segment: [install:install-file] (aggregator-style)
	[INFO] ------------------------------------------------------------------------
	[INFO] [install:install-file]
	[INFO] Installing c:\kaptcha-2.3.jar to 
	D:\maven_repo\com\google\code\kaptcha\2.3\kaptcha-2.3.jar
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESSFUL
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: < 1 second
	[INFO] Finished at: Tue May 12 13:41:42 SGT 2014
	[INFO] Final Memory: 3M/6M
	[INFO] ------------------------------------------------------------------------

	# 配置坐标
	<dependency>
	      <groupId>com.google.code</groupId>
	      <artifactId>kaptcha</artifactId>
	      <version>2.3</version>
	 </dependency>

## plugin

maven中的所有的命令都是由插件构成

	E:\MavenRepository\org\apache\maven\plugins

	# 配置maven-eclipse插件
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-eclipse-plugin</artifactId>
		<version>2.9</version>
		<configuration>
		    <!-- Always download and attach dependencies source code -->
			<downloadSources>true</downloadSources>
			<downloadJavadocs>false</downloadJavadocs>
			<!-- Avoid type mvn eclipse:eclipse -Dwtpversion=2.0 -->
			<wtpversion>2.0</wtpversion>
		</configuration>
	</plugin>

### 编译插件配置

	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-compiler-plugin</artifactId>
		<version>3.5.1</version>
		<configuration>
			<source>1.6</source>
			<target>1.6</target>
			<encoding>UTF-8</encoding>
		</configuration>
	</plugin>

### 测试插件
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-surefire-plugin</artifactId>
		<version>2.7.1</version>
		<configuration>
			<forkMode>once</forkMode>
			<argLine>-Dfile.encoding=UTF-8</argLine>
			<testFailureIgnore>true</testFailureIgnore>
		</configuration> 		
	</plugin>

### tomcat插件
	# 默认插件，无需配置
	<plugin>
		<groupId>org.codehaus.mojo</groupId>
		<artifactId>tomcat-maven-plugin</artifactId>
		<configuration>
		 <!—可选，指定端口-->
	    <port>8080</port>
			<!—默认使用8080端口，Context Path为build标签中finalName指定的名称，若没指定，则为artifactId的值-->
		</configuration>
	</plugin>

	<!-- 必须要配置后， 才能使用 -->
	<plugin>
		<groupId>org.apache.tomcat.maven</groupId>
		<artifactId>tomcat7-maven-plugin</artifactId>
		<version>2.2</version>
		<configuration>
			<port>9002</port>
		</configuration>
	</plugin>

默认： tomcat:run 
* Tomcat6插件： tomcat6:run 
* Tomcat7插件： tomcat7:run 

