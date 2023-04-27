---
title: MySql bing log 简单笔记
date: 2019-10-27 16:23:02
slug: mysql-binlog
tags:
- mysql
categories:
- 后端
---
上周工作上发生了一次"不可思议"的数据丢失问题，应该存在的数据却不见了，最后通过MySQL bin log 日志成功找了回来。之前只是听闻但没机会接触这块内容，趁着周末看了一些 bin log 日志的资料 , 包括如何将 bin log 如何用于 主从同步，记录一下。


### 什么是 bin log
bin log 也就是二进制日志(binary)，记录了数据库的所有变化。就是 insert、update、delete 这类可能会更改数据的语句，会记录其中。
而Select语句则不会记录。

由此可以看出，bin log 可以用于：
- 审查看数据库曾发生了什么事情

像我这次"数据丢失"状态，我曾怀疑是否数据没写入成功，但最后从 bin log 日志清楚看到inset记录时，从打消了我的疑问。而后一番搜索又找到了数据delete记录，这时才明确数据时被误删除了。

- 基于时间点或者pos恢复记录。

既然bin log 记录数据库所有变更操作，那如果数据丢失，自然可以从其中恢复。比如今日九点后的数据全部丢失了，那就可以执行bin log中时间点在九点后的日志，即可恢复丢失数据了。
除了基于时间点的恢复，更精确的可以基于执行点的恢复(position)。后面会讲到。

- 主从同步。和恢复数据类似，从服务器(slave)通过读取主服务器(master)的 bin log 日志，即可知道 master 上发生了什么变更，slave 再执行一遍同样的操作，即可确保两边数据完全一样。

### 启用 bin log
在MySQL的mysql.conf 中，配置 server-id 、log_bin, 重启 mysqld 即可
```
// server-id , 用于主从同步的，需要配置一个不会与其他MySQL实例重复的值即可；
// log_bin 配置基本文件名
 server-id       = 1 
 log_bin         = /var/log/mysql/mysql-bin.log
```
配置后，重启mysqld ，查看相关变量即可查看
```
mysql> show variables like "%log_bin%";
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             | // 是否开启
| log_bin_basename                | /var/log/mysql/mysql-bin       |
| log_bin_index                   | /var/log/mysql/mysql-bin.index | // index 文件，存储的是元数据。
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
```

bin log 有两种相关文件，其一是 index 文件，也就是所谓的元文件，记录了日志本身的信息，这里不深究；
另一种是日志文件，记录了所有变更信息，会以上面的`log_bin_basename`开头，后面在接上日志编号，如：
/var/log/mysql/mysql-bin.000001,/var/log/mysql/mysql-bin.000002 这样。这里我们主要关注日志文件。

### 查看日志写入状态
可通过 `show master status` 来查看日志状态。关于这个语句，MySQL文档里说的是：
> This statement provides status information about the binary log files of the master. 

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
这里看到，当前写的是 mysql-bin.000001 这个文件，写到的位置是 156 。
一般来说，bin log 会一直写一个文件，可手动使用 `flush logs` 来刷新文件(即写下一个文件)。此文，如遇MySQL重启时，也会自动执行`flush logs操作`

```
//可通过 flush 刷新日志；
mysql> flush logs;
Query OK, 0 rows affected (0.02 sec)
//刷新之后看到，已经在写 mysql-bin.000002 文件了
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

另外，可通过 reset master 来清空 bin log 日志:
```
可通过 rest master 来清空bin log 日志；
mysql> reset master;
Query OK, 0 rows affected (0.02 sec)

//file 又变成了 mysql-bin.00001
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```


### bin log 解密
知道了bin log 是怎么回事，而bin log是二进制日志，如何才能将其还原为文本查看其中内容呢？
可以用 `mysqlbinlog` 这个工具，这个工具会随MySQL自动安装，使用语法：
`mysqlbinlog mysql-bin.000029 > bin.txt` , 即可将bin日志转换成文本，并且重定向到一个 bin.txt 文本中。

### 从日志中恢复
恢复一般分为三种：
- 基于时间点的恢复：
mysqlbinlog --start-datetime="起始时间" --stop-datetime="结束时间"  /var/log/mysql/mysql-bin.000003 | mysql -uroot -p123456

- 基于日志点(Position)的恢复
mysqlbinlog --start-position=538 --stop-position=646  /var/log/mysql/mysql-bin.000003 | mysql -uroot -p123456

- 或者全部恢复，即无需填写 postion / datetime 相关参数；

至于时间点，则需要仔细审查日志才能确定。