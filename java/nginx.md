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
    * 轮询（默认）:每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除
    * 权重：指定weight值，默认1，权重越高分配的几率越大，用于后端服务器性能不均的情况
    * ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
    * fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配
    * url_hash（第三方）:按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效
```conf
# 轮询
upstream backserver { 
    server 192.168.0.14; 
    server 192.168.0.15; 
} 
# 权重
upstream backserver { 
    server 192.168.0.14 weight=8; 
    server 192.168.0.15 weight=10; 
} 
# ip_hash
upstream backserver { 
    ip_hash; 
    server 192.168.0.14:88; 
    server 192.168.0.15:80; 
} 
# fair
upstream backserver { 
    server server1; 
    server server2; 
    fair; 
} 
# url_hash
upstream backserver { 
    server squid1:3128; 
    server squid2:3128; 
    hash $request_uri; 
    hash_method crc32; 
}
```




