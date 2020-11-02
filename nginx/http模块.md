## HTTP 请求处理的11个阶段

| POST_READ 读取到header之后 | realip （获取用户真实ip）      |
| -------------------------- | ------------------------------ |
| SERVER_REWRITE             | rewrite                        |
| FIND_CONFIG location匹配   |                                |
| REWRITE                    | rewrite                        |
| POST_REWRITE               |                                |
| PREACCESS 能不能访问       | limt_conn,limit_req 限流限速   |
| ACCESS 用户访问ip          | auth_basic,access,auth_request |
| POST_ACCESS 第三方访问     |                                |
| PRECONTENT 处理content之前 | Try_files                      |
| CONTENT                    | Index,autoindex,concat         |
| LOG                        | Access_log                     |



### realip 

--with-http_realip_module启用功能(修改客户端地址)

X-Real-IP，X-Forwarded 获取用户真实ip 

基于变量使用：remote_addr,binary_remote_addr

指令:set_real_ip_from

​		set_ip_header

​		real_ip_recursive

### rewrite  重写url

#### return 指令

return code｜code url（重定向）｜URL

Error_page  将code定向到页面

```nginx
error_page 404 /404.html
```

#### 重写URL

Rewrite  regex replacement [flag]  基于匹配规则替换url

--last

--break

--redirect 302

--permanent 301

#### 条件判断

if指令

| 变量是否为空         | 直接使用       |
| -------------------- | -------------- |
| 变量与字符串是否匹配 | =，!=          |
| 变量与正则匹配       | ～ !~, ~* ,!~* |
| 文件是否存在         | -f !-f         |
| 目录是否存在         | -d, !-d        |
| 文件目录软链         | -e !-e         |
| 可执行文件           | -x !-x         |

### find_config阶段 

寻找location指令块

```nginx
location [=|~|~*|^~] uri {...}
```

前缀字符串：= 精确  ^~:匹配后不进行正则

正则：～（大小写敏感），～*

合并连续的/符号

用于内部跳转location： @



#### 匹配规则：

1.对所有location进行匹配

2.精确 > ^~ > 正则 > 最长前缀匹配



### preaccess 限制

#### limit_conn 并发连接限制 

生效范围：全部worker进程

有效性：取决于key的限制，依赖realip

|                                   |                             |
| --------------------------------- | --------------------------- |
| Limit_conn_zone key zone=namesize | 定义共享内存大小，key关键字 |
| Limit_conn zone number            | 限制并发连接数              |
| Limit_conn_log_level              | 日志发生的级别              |
| Limit_conn_status code            | 限制发生向用户发送的错误吗  |



#### limit_req 一个连接上的请求数限制

生效范围：全部worker进程（基于共享内存）

算法：leaky bucket算法(漏桶算法)

请求储存在bucket中，限制为恒定流量。bucket满了返回错误。

|                                                              |                             |
| ------------------------------------------------------------ | --------------------------- |
| limit_req_zone key zone=name:size rate=rate                  | 定义共享内存，key，限制速率 |
| Limit_req zone=name [burst=number] [nodelay(指定可这个直接返回报错)] | 限制桶的容量                |
| Limit_req_log_level                                          | 日志发生的级别              |
| Limit_red_status code                                        | 限制发生向用户发送的错误吗  |



### access阶段 

#### 对ip做限制

```
alow 允许那些地址访问 http location
deny 拒绝哪些地址访问
```

#### 对用户密码做限制 auth_basic_authentication(感觉不常用)



#### 使用第三方做权限控制 auth_request(比如我们的权限验证服务，简单了解)

收到请求，生成子请求，通过反向代理发送给上游服务器。401，403直接返回给用户



#### satisfy 允许修改模块执行顺序

```
satisfy all|any; 
```

### precontent

#### 按序访问资源模块

try_files

依次试图访问多个url对应的文件（由root或者alias指令指定），如果所有文都不存在，则按最后一个url结果或者code返回。

```nginx
location /second {
  try_files $uri $uri/index.html $uri.html =404
}
```

#### 实时拷贝流量

创造镜像流量，导流到开发环境，测试环境，对子请求返回不做处理

```nginx
mirror uri｜off #设置导流的流量
```

```nginx
mirror_request_body on|off; # 是否转发body
```

块：location

### content





