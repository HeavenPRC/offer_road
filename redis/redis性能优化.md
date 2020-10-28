## Redis内部的阻塞式操作

### redis的阻塞点

客户端：网络io，键值对增删改查操作，数据库操作

磁盘：生产RDB快照，记录AOF日志，AOF重写日志

主从节点：主库生产，传输RDB文件，从库接收RDB文件，清空数据库，加载RDB文件

切片数据库实例：向其他实例传输hash槽信息，数据迁移

#### 客户端阻塞点

网络IO：采用了IO多路复用,不会产生Io阻塞

操作阻塞：集合全量查询和聚合查询， 以及大批量数据删除内存链表回收阻塞（包括bigkey和清空数据库）。

#### 磁盘交互阻塞点

生产日志：由子进程来操作，不会产生阻塞

AOF日志：同步写模式会造成阻塞

#### 主从节点交互

从库加载RDB文件阻塞

从库FlUSHDB阻塞

#### 切片集群

Redis Cluster使用同步迁移，其中有bigkey就会造成阻塞

#### 总结阻塞点

* 集合全量查询和聚合操作；

* 从库加载 RDB 文件。

  

* bigkey 删除；

* 清空数据库；

* AOF 日志同步写；

其中

* bigkey 删除；
* 清空数据库；
* AOF 日志同步写；

这三个都可以通过异常子线程机制优化

### 异步子线程

Redis 主线程启动后，会使用操作系统提供的 pthread_create 函数创建 3 个子线程，分别由它们负责 AOF 日志写操作、键值对删除以及文件关闭的异步执行。

![avatar](https://static001.geekbang.org/resource/image/ae/69/ae004728bfe6d3771c7424e4161e7969.jpg)

异步的键值对删除和数据库清空操作是 **Redis 4.0** 后提供的功能，Redis 也提供了新的命令来执行这两个操作。
键值对删除：当你的集合类型中有大量元素（例如有百万级别或千万级别元素）需要删除时，我建议你使用 UNLINK 命令。

清空数据库：可以在 FLUSHDB 和 FLUSHALL 命令后加上 ASYNC 选项，这样就可以让后台子线程异步地清空数据库，如下所示：

```
FLUSHDB ASYNC
FLUSHALL AYSNC
```



## CPU核和NUMA架构的影响



## Redis关键系统配置

基线性能：低压力，无干扰情况下的基本性能，只由当前的软硬件决定。

```
./redis-cli --intrinsic-latency 120
Max latency so far: 17 microseconds.
Max latency so far: 44 microseconds.
Max latency so far: 94 microseconds.
Max latency so far: 110 microseconds.
Max latency so far: 119 microseconds.

36481658 total runs (avg latency: 3.2893 microseconds / 3289.32 nanoseconds per run).
Worst run took 36x longer than the average latency.
```

如果发现redis运行时延迟时基线性能的两倍，就可以认为Redis运行变慢了

Redis自身特性，操作系统，文件系统三大块进行分析

#### Redis自身特性

1.慢查询命令（注意IO，操作时间复杂度）

Redis日志，latency monitor工具

slowlog-log-slower-than设置为20000微秒,slowlog-max-len设置为1000

```
config set slowlog-log-slower-than 20000
config set slowlog-max-len 1000
config rewrite
```

如果要Redis将配置持久化到本地配置文件,需要执行config rewrite命令

虽然慢查询日志是存放在Redis内存列表中的,但是Redis并没有暴露这个列表的键,而是通过一组命令来实现对慢查询日志的访问和管理

- 获取慢查询日志 slowlog get [n]

每个慢查询日志有4个属性组成,分别是慢查询日志的标识id、发生时间戳、命令耗时、执行命令和参数

- 获取慢查询日志列表当前的长度 slowlog len
- 慢查询日志重置 slowlog reset

2.过期key

步骤：

* 采样 ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP 个数的 key，并将其中过期的 key 全部删除；
* 如果超过 25% 的 key 过期了，则重复删除的过程，直到过期 key 的比例降至 25% 以下。
  ACTIVE_EXPIRE

但是，如果触发了上面这个算法的第二条，Redis 就会一直删除以释放内存空间。注意，删除操作是阻塞的（Redis 4.0 后可以用异步线程机制来减少阻塞影响）。所以，一旦该条件触发，Redis 的线程就会一直执行删除，这样一来，就没办法正常服务其他的键值操作了，就会进一步引起其他键值操作的延迟增加，Redis 就会变慢。

所以我们要做的是避免同一时间断内产生大量过期key

#### 文件系统AOF

![avatar](https://static001.geekbang.org/resource/image/9f/a4/9f1316094001ca64c8dfca37c2c49ea4.jpg)

everysec由后台子线程执行，always不是

当主线程使用后台子线程执行了一次 fsync，需要再次把新接收的操作记录写回磁盘时，如果主线程发现上一次的 fsync 还没有执行完，那么它就会阻塞。所以，如果后台子线程执行的 fsync 频繁阻塞的话（比如 AOF 重写占用了大量的磁盘 IO 带宽），主线程也会阻塞，导致 Redis 性能变慢。

如果业务应用对延迟非常敏感，但同时允许一定量的数据丢失，那么，可以把配置项 no-appendfsync-on-rewrite 设置为 yes，如下所示：

```
no-appendfsync-on-rewrite yes
```

这个配置项设置为 yes 时，表示在 AOF 重写时，不进行 fsync 操作。也就是说，Redis 实例把写命令写到内存后，不调用后台线程进行 fsync 操作，就可以直接返回了。当然，如果此时实例发生宕机，就会导致数据丢失。反之，如果这个配置项设置为 no（也是默认配置），在 AOF 重写时，Redis 实例仍然会调用后台线程进行 fsync 操作，这就会给实例带来阻塞。

#### 操作系统： swap

内存 swap 是操作系统里将内存数据在内存和磁盘间来回换入和换出的机制，涉及到磁盘的读写，所以，一旦触发 swap，无论是被换入数据的进程，还是被换出数据的进程，其性能都会受到慢速磁盘读写的影响。

Redis 是内存数据库，内存使用量大，如果没有控制好内存的使用量，或者和其他内存需求大的应用一起运行了，就可能受到 swap 的影响，而导致性能变慢。

#### 操作系统：内存大页

Linux 内核从 2.6.38 开始支持内存大页机制，该机制支持 2MB 大小的内存页分配，而常规的内存页分配是按 4KB 的粒度来执行的。

虽然内存大页可以给 Redis 带来内存分配方面的收益，

但是，Redis 为了提供数据可靠性保证，需要将数据做持久化保存。这个写入过程由额外的线程执行，

所以，此时，Redis 主线程仍然可以接收客户端写请求。客户端的写请求可能会修改正在进行持久化的数据。

在这一过程中，Redis 就会采用写时复制机制，也就是说，一旦有数据要被修改，Redis 并不会直接修改内存中的数据，而是将这些数据拷贝一份，然后再进行修改。

```
cat /sys/kernel/mm/transparent_hugepage/enabled
```

```
echo never /sys/kernel/mm/transparent_hugepage/enabled
```

## Redis内存碎片

数据删除后Redis释放的内存空间会由内存分配器管理，并不会立即返回给操作系统。所以操作系统仍会记录着给Redis分配了大量内存。

#### 内存碎片

内存空间闲置，往往是因为操作系统发生了较为严重的内存碎片。

其实是连续的空间很小，总空间大但都是碎片化的。

#### 内存碎片形成的原因

* 内存分配器

内存分配器的分配策略就决定了操作系统无法做到“按需分配”。这是因为，内存分配器一般是按固定大小来分配内存，而不是完全按照应用程序申请的内存空间大小给程序分配。一般是分配比需求略大的固定内存空间，以此来减少分配次数。

* redis负载特征

键值对大小不一致

![avatar](https://static001.geekbang.org/resource/image/46/a5/46d93f2ef50a7f6f91812d0c21ebd6a5.jpg)

键值对会被修改和删除

这样会导致空间的扩容和释放

![avatar](https://static001.geekbang.org/resource/image/4d/b8/4d5265c6a38d1839bf4943918f6b6db8.jpg)

#### 如何判断是否有内存碎片

```
Info memory
```

``` 
# Memory
# 由 Redis 分配器分配的内存总量，包含了redis进程内部的开销和数据占用的内存，以字节（byte）为单位（是你的Redis实例中所有key及其value占用的内存大小）    
used_memory:1073741736   
used_memory_human:1024.00M 
# 向操作系统申请的内存大小
used_memory_rss:1997159792
used_memory_rss_human:1.86G
# redis的内存消耗峰值(以字节为单位)
used_memory_peak:458320936  
used_memory_peak_human:437.09M  
# 系统内存总量
total_system_memory:33614647296    
total_system_memory_human:31.31G  
# Lua脚本存储占用的内存
used_memory_lua:37888   
used_memory_lua_human:37.00K   
# Redis实例的最大内存配置（设置的最大内存）
maxmemory:7000000000   
maxmemory_human:6.52G  
maxmemory_policy:noeviction  # 当达到maxmemory时的淘汰策略
mem_fragmentation_ratio:1.86 # 内存碎片比例
```

```
mem_fragmentation_ratio = used_memory_rss/ used_memory
```

##### mem_fragmentation_ratio < 1.5 正常

##### mem_fragmentation_ratio > 1.5 不正常

#### 清理内存碎片 

4.0-RC3之前，暴力重启。数据恢复需要时间

4.0之后

![avatar](https://static001.geekbang.org/resource/image/64/42/6480b6af5b2423b271ef3fb59f555842.jpg)

首先，Redis 需要启用自动内存碎片清理，可以把 activedefrag 配置项设置为 yes，命令如下：

````mysql
config set activedefrag yes
````

这个命令只是启用了自动清理功能，但是，具体什么时候清理，会受到下面这两个参数的控制。这两个参数分别设置了触发内存清理的一个条件，如果同时满足这两个条件，就开始清理。在清理的过程中，只要有一个条件不满足了，就停止自动清理。

* active-defrag-ignore-bytes 100mb：表示内存碎片的字节数达到 100MB 时，开始清理；

* active-defrag-threshold-lower 10：表示内存碎片空间占操作系统分配给 Redis 的总空间比例达到 10% 时，开始清理。

为了尽可能减少碎片清理对 Redis 正常请求处理的影响，自动内存碎片清理功能在执行时，还会监控清理操作占用的 CPU 时间，而且还设置了两个参数，分别用于控制清理操作占的 CPU 时间比例的上、下限，既保证清理工作能正常进行，又避免了降低 Redis 性能。这两个参数具体如下：

* active-defrag-cycle-min 25： 表示自动清理过程所用 CPU 时间的比例不低于 25%，保证清理能正常开展；
* active-defrag-cycle-max 75：表示自动清理过程所用 CPU 时间的比例不高于 75%，一旦超过，就停止清理，从而避免在清理时，大量的内存拷贝阻塞 Redis，导致响应延迟升高。

## Redis 缓冲区

* 用一块内存空间来暂时存放**命令数据**,以免出现因为数据和命令的处理速度慢于发送速度而导致数据丢失和性能问题。
* 主从节点同步时，用来暂存主节点接收的写命令和操作

**产生的问题：**

大小有限，占用内存超过阈值，就会出现缓冲区溢出，严重还会导致实例崩溃。

### 客户端输入输出缓冲区

![avatar](https://static001.geekbang.org/resource/image/b8/e4/b86be61e91bd7ca207989c220991fce4.jpg)

造成缓冲区溢出的原因：

* 写入了 bigkey，比如一下子写入了多个百万级别的集合类型数据；
* 服务器端处理请求的速度过慢，例如，Redis 主线程出现了间歇性阻塞，无法及时处理正常发送的请求，导致客户端发送的请求在缓冲区越积越多。

#### 查看缓冲区使用情况

```
CLIENT LIST
id=5 addr=127.0.0.1:50487 fd=9 name= age=4 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client
```

* cmd，表示客户端最新执行的命令。这个例子中执行的是 CLIENT 命令。
* qbuf，表示输入缓冲区已经使用的大小。这个例子中的 CLIENT 命令已使用了 26 字节大小的缓冲区。
* qbuf-free，表示输入缓冲区尚未使用的大小。这个例子中的 CLIENT 命令还可以使用 32742 字节的缓冲区。qbuf 和 qbuf-free 的总和就是，Redis 服务器端当前为已连接的这个客户端分配的缓冲区总大小。这个例子中总共分配了 26 + 32742 = 32768 字节，也就是 32KB 的缓冲区。
  

通常情况下，Redis 服务器端不止服务一个客户端，当多个客户端连接占用的内存总量，超过了 Redis 的 maxmemory 配置项时（例如 4GB），就会触发 Redis 进行数据淘汰。一旦数据被淘汰出 Redis，再要访问这部分数据，就需要去后端数据库读取，这就降低了业务应用的访问性能。此外，更糟糕的是，如果使用多个客户端，导致 Redis 内存占用过大，也会导致内存溢出（out-of-memory）问题，进而会引起 Redis 崩溃，给业务应用造成严重影响。

#### 原因

* 服务器返回bigkey的大量结果
* 执行了MONITOR命令
* 缓冲区大小设置不合理

MONITOR 命令是用来监测 Redis 执行的。执行这个命令之后，就会持续输出监测到的各个命令操作，如下所示：

```
MONITOR
OK
1600617456.437129 [0 127.0.0.1:50487] "COMMAND"
1600617477.289667 [0 127.0.0.1:50487] "info" "memory"
```

#### 普通客户端（命令客户端）：

因为是阻塞式发送所以不做限制

```
client-output-buffer-limit normal 0 0 0
```

normal 表示当前设置的是普通客户端，

第 1 个 0 设置的是缓冲区大小限制，

第 2 个 0 和第 3 个 0 分别表示缓冲区持续写入量限制和持续写入时间限制。

#### 订阅客户端：

```
client-output-buffer-limit pubsub 8mb 2mb 60
```

### 主从集群中的缓冲区

主从集群间的数据复制包括**全量复制**和**增量复制**，

为了保存主从一致都会用到缓冲区

#### 全量复制

```
config set client-output-buffer-limit slave 512mb 128mb 60
```

其中，slave 参数表明该配置项是针对复制缓冲区的。512mb 代表将缓冲区大小的上限设置为 512MB；128mb 和 60 代表的设置是，如果连续 60 秒内的写入量超过 128MB 的话，也会触发缓冲区溢出。

#### 增量复制-复制积压缓冲区repl_backlog_buffer

repl_backlog_size



