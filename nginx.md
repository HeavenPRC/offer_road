## 1.安装

nginx.org 下载需要的稳定版本

```
wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar -xzf nginx-1.18.0.tar.gz
cp -r contrib/vim/* ~/.vim/ # 可以让我们的vim识别conf
./configure --help # 查看参数
./configure --prefix=/usr/local/nginx #生成objs文件夹
make
make install
```

解压各文件夹含义

```
auto   编译辅助编辑

changes  版本历史

Conf 示例文件

configure 前置编译工具

contrib vim工具配置

html 500 index.com

Man 帮助文件

src 源代码
```

configure 支持的参数

```
1. 使用模块的路径 安装的录像 辅助文件的路径
2. 使用那些模块不使用哪些。 写with的通常不会主动编译 写with-out的默认会编译
3. 特殊参数 
```



## 2.conf配置语法

1.配置文件由指令与指令块构成

2.每条指令以；结尾，指令与参数间以空格符号分隔

```nginx
worker_processes  1;
```

3.指令块以{}大括号将多条指令组织在一起

```nginx
events {
    worker_connections  1024;
}
```

4.include语句允许组合多个废纸文件以提升可维护性

```nginx
 include       mime.types;
```

5.使用#符号添加注释，提高可读性

```nginx
#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
```

6.使用$符号使用变量

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
```

7.部分指令的参数支持正则表达式



时间的单位

| 单位 | 描述 |
| ---- | ---- |
| ms   | 毫秒 |
| s    | 秒   |
| m    | 分钟 |
| h    | 小时 |
| d    | 天   |
| w    | 周   |
| M    | 月   |
| y    | 年   |

空间单位

| bytes |      |
| ----- | ---- |
| k/K   |      |
| m/M   |      |
| g/G   |      |

http 配置的指令块  -- http ｜server ｜location ｜upstream 

## 3.访问日志access_log

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
access_log  logs/host.access.log  main;
```

 log_format 日志格式   main  日志格式的命名

access_log  指定日志位置 和使用的日志格式



## 4.反向代理以及缓存

```nginx
# 上游服务
upstream local {
  server 127.0.0.1:8080;
  server 127.0.0.1:8081;
} 
# 缓存 nginx的性能远远优于上游服务器 不过我觉得还是各自分工明确的好 容易维护
proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m 
							use_temp_path=off; 
server {
  server_name op.starpike.cn
  listen 80;
  	
  location / {
    # 因为是反向代理 中间隔着一层 有些header上游服务端会拿不到 通过proxy_set_header向上游传输真实的header信息
    # 相关模块:ngx_http_proxy_module http://nginx.org/en/docs/http/ngx_http_proxy_module.html
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   	
    proxy_cache my_cache;
    
    proxy_cache_key $host$url$is_args$args;
    proxy_cache_valid 200 304 302 ld;
    
    proxy_pass http://local
  }
}
```



## 5.https站点

安装证书生成工具

```shell
yum install python2-certbot-nginx
```

生成证书

```
certbot --nginx --nginx-server-root=/search/soft/nginx/conf/ -d service.starpike.com
```

nginx对ssl的优化

```nginx
ssl_session_cahe shared:le_nginx_SSL:1m; # 对握手进行缓存
ssl_session_time_out 1440m; # 握手之后断开在时间以内无需再次

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;

ssl_ciphers xxxx; # 安全套件
ssl_dhparam ; 
```



## 6.nginx 进程管理信号

| Master进程                                                   | Worker进程                                              | Nginx命令行                                                |
| ------------------------------------------------------------ | ------------------------------------------------------- | ---------------------------------------------------------- |
| 监控worker进程 CHLD                                          |                                                         |                                                            |
| 管理worker进程                                               |                                                         |                                                            |
| 接收信号：<br />TERM,INT<br />QUIT<br />HUP<br />USR1<br />USR2<br />WINCH | 接收信号：<br />TERM,INT<br />QUIT<br />USR1<br />WINCH | Reload:HUP<br />reopen:USER1<br />stop:TERM<br />quit:QUIT |



## 7.优化实践

```nginx
# worker 进程和cpu核数保持一致
worker_processes  2;

# 连接池包含 上游服务器和客户端的连接
# 一个连接 结构体占232byte + 两个读写事件结构体96 * 2 = 424byte
events{
      worker_connections 512;
}

# 内存池提前分配好 请求内存池 512 连接内存池 4k  一般不做修改
Syntax: connection_pool_size size;
Default: connection_pool_size 256|512;
Context: http, server

Syntax: request_pool_size size;
Default: request_pool_size 256|512;
Context: http, server
```



## nginx relod的流程

向master进程发送HUP信号

master进程校验配置语法是否正确

master进程打开新的监听端口

master进程用新配置启动新的worker子进程

master进程向老worker子进程发送QUIT信号 （有请求还会存在，在timeout后直接停止）

老worker进程关闭监听句柄，处理完当前连接后结束进程



## nginx 热升级 

将旧nginx文换成新的nginx文件 （备份）

向master进程发送USR2信号 kill -USR2 [master进程的pid]

master进程修改pid文件名，加后缀.oldbin

master进程用新Nginx文件启动新master进程

正常:向老msater进程发送QUIT信号，关闭老master进程

回滚:向老master发送HUP，向新msater发送QUIT



## nginx 优雅关闭QUIT 

识别进程中没有处理请求----http请求

1.设置定时器 worker_shutdown_timeout

2.关闭监听句柄

3.关闭空闲连接

4.在循环中等待全部连接关闭

5.退出进程



## nginx 基础指令

```
nginx
-? -h 帮助
-c 使用指定的配置文件
-g 指定配置指令
-p 指定运行目录
-s 发送信号 stop立即停止 quit优雅的停止服务 reload重载 reopen 重新开始记录日志文件
-t -T 测试配置文件是否语法错误
-v -V 打印nginx的版本信息，编译信息等
```



### nginx日志切割 reopen



## 日志可视化

goaccess /search/soft/nginx/logs/service.access.log -o /search/soft/nginx/html/report.html  --real-time-html  --time-format='%H:%M:%S'  --date-format='%d/%b/%Y' --log-format=COMBINED

goaccess -a -d -f /search/soft/nginx/logs/service.access.log -p /usr/local/goaccess/etc/goaccess/goaccess.conf -o /search/soft/nginx/html/report.html --real-time-html --daemonize



```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

