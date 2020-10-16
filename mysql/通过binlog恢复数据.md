# [Mysql 通过binlog日志恢复数据](https://www.cnblogs.com/YCcc/p/10825870.html)

Binlog日志，即binary log，是二进制日志文件，有两个作用，一个是增量备份，另一个是主从复制，即主节点维护一个binlog日志文件，从节点从binlog中同步数据，也可以通过binlog日志来恢复数据

1，登录mysql查看binlog日志的状态，输入show variables like ‘%log_bin%’;查看binlog为off关闭状态

2，开启mysql binlog日志，进入mysql配置文件(vi /etc/my.cnf) 在mysqld区域内添加如下内容，①server-id = 1(单个节点id) ②log-bin= /var/lib/mysql/mysql-bin(位置一般和mysql库文件所在位置一样) ③expire_logs_days = 10(表示此日志保存时间为10天),重启mysqld,再次查看binlog日志开启状态为ON

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507152908222-639253368.png)

3，Binlog日志包括两类文件；第一个是二进制索引文件(后缀名为.index),第二个为日志文件(后缀名为.00000*),记录数据库所有的DDL和DML(除了查询语句select)语句事件

4，查看所有binlog日志文件列表:show master logs;

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507152940162-384506609.png)

5，查看最后一个binlog日志的编号名称及其最后一个操作事件pos结束点的值：show master status; 

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153015604-102688683.png)

6，Flush logs 刷新日志，此刻开始产生一个新编号的binlog文件，例如：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153048055-1231513603.png)

每当mysqld服务重启时，会自动执行刷新binlog日志命令，mysqldump备份数据时加-F选项也会刷新binlog日志

7，清空所有binlog日志命令:reset master;

8，查看binlog文件内容，使用查看工具mysqlbinlog来查看(cat/vi/more都是无法打开的) 

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153142027-1169479301.png)

9，上面的方法读取内容较多不易观察，以下命令更为方便查看命令：show binlog events in ‘mysql-bin.000003’; 

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153210783-2109343465.png)

10，指定查询,从pos点406开始查询，如下：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153243655-1493589710.png)

11，指定查询，从pos点154开始查询，中间跳过2行，查询4条数据，如图：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153310115-1117770193.png)

12，利用binlog日志恢复mysql数据，例如，现有一张数据表如图：此表位于hello数据库

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153336320-433713266.png)

现在将此数据库备份到本地(模拟每周的备份情况)，备份命令如下

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153355285-713036786.png)

可以在数据备份之前或者之后执行flush logs重新生成一个binlog日志用来记录备份之后的所有增删改操作(重新生成日志更好找pos点)，由于业务需求，现在对表进行插入

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153407206-835543014.png)

数据，修改数据，如图新增了两条数据id 为5和6，修改后数据如图，

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153424967-1016954983.png)

6的年龄修改成了30

13，由于操作失误，误删除了数据库，所有数据都不见了,此时可以通过binlog日志恢复数据，由于之前有做了数据库备份，所以可以先将备份的数据导入进去，剩下缺少的就是备份之后操作所产生的内容(备份之后执行了插入内容以及更改内容)，先恢复备份的数据：创建库hello并选择库(use hello)，通过命令source /root/hello.sql将数据库内容导入，然后执行

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153459162-146239595.png)

14，恢复的内容如下所示：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153520755-1685126953.png)

15，上面恢复的数据只是截止到备份时间的数据，剩下缺少的数据可以通过binlog日志来恢复，由于我们备份数据库之前重新创建了mysql-bin.000006日志，所以备份后的所有操作都保存在这个日志中，可以先备份下这个日志文件，cp mysql-bin.000006 /root/  接着执行flush logs 刷新日志，重新创建了一个binlog日志7

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153705992-1642719490.png)

16，重新创建一个日志7的目的：接下来所有操作的数据都会写入到日志7中，日志6中不会在写入任何数据(方便根据日志6的内容恢复数据，因为日志6的数据就是备份之后到删库之前的所有操作日志，重建日志7不会有过多的数据影响恢复)

### 指定pos点恢复

17，登录mysql 查看日志6: show binlog events in ‘mysql-bin.000006’;结果如下：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153747453-302110365.png)

查看日志发现，在备份数据后首先执行的是插入数据操作(在一个事务内)，此时可以通过指定pos结束点恢复(部分恢复)，如图，pos结束点位435(事务已提交表示结束)，执行命令如下, 

**/usr/bin/mysqlbinlog --stop-position=435 --database=hello /var/lib/mysql/mysql-bin.000006 | /usr/bin/mysql -uroot -p密码 -v hello** 

(其中整个命令的含义是通过mysqlbinlog读取日志内容并通过管道传给mysql命令，-v表示执行此mysql命令)执行后查询表发现如下：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153803250-24072693.png)

备份之后插入的数据都出现了，目前还差一次修改的数据没有找回来，接着看日志6如图，

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153815178-951950790.png)

发现执行更新操作的事务区间为573到718，所以可以执行以下命令来恢复这段数据，

**/usr/bin/mysqlbinlog --start-position=573 --stop-position=718 --database=hello /var/lib/mysql/mysql-bin.000006 | /usr/bin/mysql -uroot -p密码 -v hello**

注意其中的符号都是英文状态下(只恢复这段事务区间的数据也就是更新的数据)，恢复结果如下：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153829139-1493667044.png)

更改的数据回来了，也可以最开始的时候直接通过最终的pos结束点718来恢复

18，还有一种通过时间来恢复，还是先备份当前数据库(备份之前先flush logs创建一个新的binlog日志文件9)，然后修改数据插入数据后如图，

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153858656-808203229.png)

修改后将当前数据库删除，(删除库之后再执行一下flush logs创建一个新的日志文件，这样新的操作都写入到这个文件中，上个日志文件只用来恢复数据，不会有其余数据混入)现在我们通过时间点来恢复从备份数据后到删除数据库这期间所有操作的数据，首先通过mysqlbinlog来了查看日志文件9如图所示，

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153914490-1322242918.png)

### 根据开始结束时间进行恢复

19，执行命令：

**/usr/bin/mysqlbinlog --start-datetime="2018-04-27 20:57:55" --stop-datetime="2018-04-27 20:58:18" --database=hello /var/lib/mysql/mysql-bin.000009 | /usr/bin/mysql -uroot -p8856769abcd -v hello **更改的数据得到了恢复

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153932470-727000297.png)

20，插入数据的日志时间如图：

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153946569-1830102727.png)

执行命令/usr/bin/mysqlbinlog --start-datetime="2018-04-27 20:58:18" --stop-datetime="2018-04-27 20:58:35" --database=hello /var/lib/mysql/mysql-bin.000009 | /usr/bin/mysql -uroot -p8856769abcd -v hello 插入的数据得到了恢复，如图，

![img](https://img2018.cnblogs.com/blog/1292627/201905/1292627-20190507153957144-2056740859.png)