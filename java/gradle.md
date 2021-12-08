# project

* 一个 Gradle 构建都由一个或者多个project 组成,每个 project 包括许多的构建部分，可以是一个 jar 包，也可以是一个 web 应用，也可以是多个 jar 的整合，可以部署应用和搭建环境
* 一个 project 代表一个正在构建的组件(jar/war文件)，当构建开始时，Gradle 会基于 build.gradle 实例化一个` org.gradle.api.Project` 对象，并通过 project 变量来隐式调用其成员

# task

* 每个任务在构建执行过程中会被封装成 `org.gradle.api.Task` 对象. 主要包括任务的动作和任务依赖.任务动作定义了一个原子操作.可以定义依赖其他任务、动作顺序和执行条件。

* 任务主要操作动作

  * dependsOn : 依赖相关操作
  * doFirst ： 任务执行之前执行的方法
  * doLast, << ： 任务执行之后执行的方法

  ```markdown
  task t1{
     # 调用task、依赖时执行
      doFirst {
          println "t1 dofirst"
      }
     # 调用project时执行 
      println "t1---"
     # 调用task、依赖时执行
      doLast {
          println "t1 dolast"
      }
  }
  # 依赖task t1
  task t2(dependsOn:'t1'){
     # 调用task、依赖时执行
      doFirst {
          println "t2 dofirst"
      }
     # 调用project时执行
      println "t2---"
     # 调用task、依赖时执行
      doLast {
          println "t2 dolast"
      }
  }
  
  # 调用task t2时，也会调用task t1的doFirst、doLast
  # 调用project时，只会执行t1、t2的println语句不会执行doFirst、doLast
  ```

* 任务依赖的方式

  * 定义任务时参数依赖 (dependsOn:'taskName')
  * 任务内部依赖 dependsOn 'taskName'
  * 外部添加依赖  taskName1.dependsOn  taskName2

# 声明周期

* 初始化阶段
  * 通过settings.gradle获取需要初始化的项目，类似maven的module配置
  * 加载所有项目下的build.gradle文件
  * 为项目创建project对象
* 配置阶段
  * 执行build.gradle脚本，完成project配置
  * 构造task依赖关系
* 执行阶段
  * 执行task中的代码
  * doFirst/doLast

# 依赖

```markdown
repositories {
   # 自定义仓库源
    maven { url 'https://maven.aliyun.com/repository/public/' }
   # 使用本地maven仓库
    mavenLocal()
   # gradle没有自己仓库，配置maven作为中央仓库 
    mavenCentral()
   # 自定义仓库源
    maven {
        url "https://repo.spring.io/libs-milestone"
    }
   # 自定义仓库源
    maven {
        url "http://repo.hualala.com/nexus/content/repositories/snapshots/"
    }
}
# 配置多个仓库时，查找按照顺，找到则返回
```

# scope

* implementation：默认scope，当前项目生效编译、运行时生效，不会暴露给其它项目
* api：当前项目生效编译、运行时生效，可以暴露给其它项目
* compileOnly：编译时可见
* runtimeOnly：运行时可以见
* testImplementation：测试编译时、测试运行时可见
* testCompileOnly：测试编译时可见
* testRuntimeOnly：测试运行时可见

# 冲突

* 排除依赖

  ```markdown
  dependencies {
      compile (group: 'org.hibernate', name: 'hibernate-core', version: '3.6.3.Final'){
      		# module 是 jar 的 name
          exclude group:"org.slf4j" , module:"slf4j-api"
      }
  }
   
  ```

* 排除jar的依赖性，不推荐

  ```markdown
  dependencies {
      compile (group: 'org.hibernate', name: 'hibernate-core', version: '3.6.3.Final'){
          transitive=false
      }
  }
  ```

* 指定jar使用版本

  ```markdown
  configurations.all{
      resolutionStrategy{
          force 'org.slf4j:slf4j-api:1.7.24'
      }
  }
  ```

* 对所有jar不做冲突解决，编译时直接报错，可以查看冲突的jar

  ```markdown
  configurations.all{
    resolutionStrategy{
        # 修改 gradle不自动处理版本冲突
        failOnVersionConflict()
    }
  }
  ```

  

# 路径信息

```markdown
# gradle包
/Users/admin/.gradle/wrapper/dists
# jar保存位置
/Users/admin/.gradle/caches/modules-2/files-2.1
```

# gradle-wrapper.properties

```markdown

distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-6.9.1-all.zip
```

# build.gradle

```markdown
# 配置运行构建脚本的要求
buildscript { 
   # 设置自定义属性
    ext {  
       springBootVersion = '2.1.6.RELEASE' 
    }  
   # 解决buildscript块中的依赖项时，检查Maven Central中的依赖项
    repositories {  
       mavenCentral()  
    }  
   # 我们需要spring boot插件来运行构建脚本
    dependencies {  
       classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")  
    }  
}  
   
# 添加构建插件
apply plugin: 'java' 
apply plugin: 'org.springframework.boot' 
apply plugin: 'io.spring.dependency-management' 
   
# 设置全局变量
group = 'com.okta.springboottokenauth' 
version = '0.0.1-SNAPSHOT' 
sourceCompatibility = 1.8 
   
# 用于搜索以解决项目依赖关系的仓库地址
repositories {  
   # 使用maven仓库
    mavenCentral()  
}  
 
# 项目依赖
dependencies {  
    implementation('com.okta.spring:okta-spring-boot-starter:1.2.1')  
    implementation('org.springframework.boot:spring-boot-starter-security')  
    implementation('org.springframework.boot:spring-boot-starter-web')  
    testImplementation('org.springframework.boot:spring-boot-starter-test')  
    testImplementation('org.springframework.security:spring-security-test')  
}
```

# Grade Wrapper

```markdown
# 生成gradle wrapper
> gradle wrapper
# 项目目录结构
> tree gradle                                                                                                                            
gradle
└── wrapper
    ├── gradle-wrapper.jar
    └── gradle-wrapper.properties

1 directory, 2 files
```

# 常用命令

```markdown
# build.gradl定义的所有tasks
gradle tasks
# build.gradl定义的所有依赖
gradle dependencies
# build.gradl定义的所有属性
gradle properties
# 构建项目，跳过测试
gradle build -x test
```



