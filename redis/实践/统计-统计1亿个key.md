### 聚合统计

多个集合元素的聚合结果；

* 交集统计
* 查集统计
* 并集统计

set的集合运算很复杂，在数据量大时容易导致redis实例的阻塞，可以选择在从库上进行后台聚合计算，或者由客户端进行计算。

### 排序统计

List：按照进入顺序排序

sorted set: 按照指定的权重排序，分页好用

```
ZRANGEBYSCORE comments N-9 N
```

### 二值状态统计

例如：签到（1） 未签到 （0）

##### bitmap redis提供的扩展数据类型

```
SETBIT uid:sign:3000:202008 2 1 
```

```
GETBIT uid:sign:3000:202008 2 
```

bitmap支持用bitop 对多个BITmap命令做按位”与“”或“”异或“，并把结果放在一个新的bitmap中

bitcount 统计bitmap中1或0的个数

### 基数统计

统计一个集合中不重复的元素个数

数据量小可以用hash，set

数据量大时就要用到**hyperLoglog**

HyperLogLog 是一种用于统计基数的数据集合类型，它的最大优势就在于，当集合元素数量非常多时，它计算基数所需的空间总是固定的，而且还很小。
在 Redis 中，每个 HyperLogLog 只需要花费 12 KB 内存，就可以计算接近 2^64 个元素的基数。和元素越多就越耗费内存的 Set 和 Hash 类型相比，HyperLogLog 就非常节省空间。
在统计 UV 时，你可以用 PFADD 命令（用于向 HyperLogLog 中添加新元素）把访问页面的每个用户都添加到 HyperLogLog 中。

```
PFADD page1:uv user1 user2 user3 user4 user5
```

统计

```
PFCOUNT page1:uv
```



