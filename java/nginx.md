## 安装

```shell
# 检查环境
$ ./configure
# 安装
$ make && make install
```

## 命令

```shell
# 启动nginx
nginx   
# 快速关闭Nginx，可能不保存相关信息，并迅速终止web服务
nginx -s stop       
# 平稳关闭Nginx，保存相关信息，有安排的结束web服务
nginx -s quit       
# 因改变了Nginx相关配置，需要重新加载配置而重载
nginx -s reload     
# 重新打开日志文件
nginx -s reopen     
# 为 Nginx 指定一个配置文件，来代替缺省的
nginx -c filename   
# 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件
nginx -t            
# 显示 nginx 的版本
nginx -v
# 显示 nginx 的版本，编译器版本和配置参数
nginx -V            
```

## 配置文件结构
```conf
全局{
    events{

    }
    http{
        http全局{

        }
        server{

        }
    }
}
```

## 反向代理
### 代理一个节点
```conf
server {
    listen       80;  # 监听的端口
    server_name  node5;  # nginx节点名称，访问时需要配置名称到ip的映射

    location / {
        root   html;
        proxy_pass http://www.baidu.com;   # 配置反向代理地址
        index  index.html index.htm;
    }
}
```
### 不同地址代理节点
```conf
server {
    listen       9000; # 监听端口
    server_name  node5; # nginx节点名称

    location / {
        root   html;
        proxy_pass http://www.baidu.com;
        index  index.html index.htm;
    }
    # 访问node5:9000/sohu/跳转到sohu
    location ~ /sohu/ { 
        proxy_pass http://www.sohu.com;
    }
    # 访问node5:9000/sina/跳转到sina
    location ~ /sina/ {
        proxy_pass http://www.sina.com;
    }
}
```
### 同一个地址代理多节点负载
```conf
# 负载均的衡节点
upstream balance{
    server node5:8080; # tomcat地址
    server node4:8080;
}

server {
    listen       9000;
    server_name  node5;
    # 配置反向代理节点
    location / {
        root   html;
        proxy_pass http://balance; # 调用upstream的配置的节点
        index  index.html index.htm;
    }
}
```
* 页面访问http://node5:9000/edu/a.html，在node4、node5两个节点轮询访问
* 负载策略
    * 轮询（默认）:每个请求按照时间顺序轮询分配，如果后端服务器down将被删除
    * weight：默认值1，权重越高分配的几率越大
    * ip_hash：请求按照ip哈希分配
    * fair：按服务器响应时间分配，响应时间短分配优先



