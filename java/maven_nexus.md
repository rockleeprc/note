*下载*

https://www.sonatype.com/download-oss-sonatype 

*版本*

`nexus-2.14.8-01`

*配置修改*

`/usr/local/nexus/nexus-2.14.8-01/bin/nexus`

`RUN_AS_USER=root`

`/usr/local/nexus/nexus-2.14.8-01/conf/nexus.properties`

*启动*

`/usr/local/nexus/nexus-2.14.8-01/bin# ./nexus start`

*访问地址*

http://ip:8081/nexus

*登录*

用户名 admin 默认密码 admin123



nexus 下载的jar保存位置

`/usr/local/nexus/sonatype-work/nexus/storage`