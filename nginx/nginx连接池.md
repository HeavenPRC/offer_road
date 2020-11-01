### 连接池配置

```
worker_connections number;
```

这个数字代表着客户端和反向代理的连接

这个大小受内存限制

基础结构体大概232bytes

读写事件结构体96*2bytes

需要按照已有内存和使用来进行配置



nginx每个worker进程都有一个独立的nginx_cycle_s

```nginx
//文件名ngx_cycle.h
struct ngx_cycle_s {
    void                  ****conf_ctx;
    ngx_pool_t               *pool;

    ngx_log_t                *log;
    ngx_log_t                 new_log;

    ngx_uint_t                log_use_stderr;  /* unsigned  log_use_stderr:1; */

    ngx_connection_t        **files;
    //指向下一个空闲连接，归还连接的 时候，只需要把该连接插入到free_connections链表表头即可
    ngx_connection_t         *free_connections;
    ngx_uint_t                free_connection_n;

    ngx_module_t            **modules;
    ngx_uint_t                modules_n;
    ngx_uint_t                modules_used;    /* unsigned  modules_used:1; */

    ngx_queue_t               reusable_connections_queue;

    ngx_array_t               listening;
    ngx_array_t               paths;
    ngx_array_t               config_dump;
    ngx_list_t                open_files;
    ngx_list_t                shared_memory;

    ngx_uint_t                connection_n;
    ngx_uint_t                files_n;
    // 指向整个连接池数组的首部
    ngx_connection_t         *connections;
    ngx_event_t              *read_events;
    ngx_event_t              *write_events;

    ngx_cycle_t              *old_cycle;

    ngx_str_t                 conf_file;
    ngx_str_t                 conf_param;
    ngx_str_t                 conf_prefix;
    ngx_str_t                 prefix;
    ngx_str_t                 lock_file;
    ngx_str_t                 hostname;
};
```



EPoll https://www.cnblogs.com/yimuren/p/4105124.html

### 内存池 一般不需要修改，对减少内存碎片有一定效果

连接内存池，

请求内存池 4k 

