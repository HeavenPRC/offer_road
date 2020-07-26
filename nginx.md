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

### nginx升级直接替换 sbin中的二进制文件 

kill -USR2 pid   

kill -WINCH 13195 优雅关闭进程

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

