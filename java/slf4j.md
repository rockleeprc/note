

### slf4j

* SLF4J(Simple Logging Facade for Java)，一套日志框架API，没有具体实现
* SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统
* SLF4J提供 TRACE, DEBUG, INFO, WARN, ERROR五种级别

### common-logging

* apache提供的日志API，自带SimpleLogger作为实现，功能比较薄弱，通常配合log4j使用
* common-logging的动态查找机制，在程序运行时找出使用的日志实现，如log4j，最差的情况就是使用SimpleLogger

### log4j
* apache的一个开放源代码项目,实现了输出到控制台、文件、 回滚文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能,且配置比较简单

### logback
* log4j创始人设计的又一个开源日志组件
* logback当前分成三个模块：logback-core,logback- classic和logback-access，logback-classic是log4j的一个 改良版本