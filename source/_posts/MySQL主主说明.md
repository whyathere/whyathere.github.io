---
title: MySQL多主循环 使用
tags: MySQL
categories: 数据库
---
### 配置文件样例：
164：
port=3306
server-id=164        
log-bin=mysql
binlog-do-db=mytest
binlog-ignore-db=mysql  #忽略的数据库
log-slave-updates=true	#需要
replicate-do-db=mytest	#要同步的数据库，如果需要多个库，重复这行
replicate-do-db=zero_bos #
slave-skip-errors=all   #忽略同步过程中的错误

### 参数说明
MySQL的配置文件各种参数说明
binlog-do-db：指定mysql的binlog日志记录哪个db
Replicate_Do_DB：参数是在slave上配置，指定slave要复制哪个库
binlog-do-db=需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可
binlog-ignore-db=不需要复制的数据库苦命，如果复制多个数据库，重复设置这个选项即可
replicate-do-db=需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可
replicate-ignore-db=需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可
log-bin=mysql-bin  //启动二进制日志系统 
sync_binlog = 1 or N：This makes MySQL synchronize the binary log’s contents to disk each time it commits a transaction 


注意：如果你想做一个复杂点的结构：比如说，A->B->C，其中B是A的从服务器，同时B又是C的主服务器，那么B服务器除了需要打开log-bin之外，还需要打开log-slave-updates选项，你可以再B上使用“show variables like 'log%';”来确认是否已经生效。

### 注意事项
1. 若同步某一数据库，应确保主从服务器上均有此数据库
2. 不指定‘binlog-do-db’ 与‘replicate-do-db’即位同步整个服务器
3. 配置完成后，尽量不要对从库进行插入、更新等操作，只进行查询操作即可

### 常用操作命令
1. 停止从库服务：slave stop;
2. 重置从库服务：slave reset;   // 重新启动从库服务后，会从新同步

#### 权限相关
1. 在主服务器上查询当前二进制文件的文件名及偏移位置：show master status;
2. 启动从服务器上的复制线程：start slave;
3. 刷新权限：FLUSH PRIVILEGES;


#### 日志相关
1. 是否启用了日志：show variables like 'log_%'; 
2. 刷新日志：flush logs;


### 从库
从库更改要复制的主库：change master to master_host='10.0.0.164',master_user='replicate',master_password='123456'，
master_log_file='mysql-bin.000013',master_log_pos=107;;



### 遇到的错误

#### 1
1.错误提示：
Last_IO_Error: Got fatal error 1236 from master when reading dat
a from binary log: 'Could not find first log file name in binary log index file'
解决方法：查看master的错误日志
```bash
180413 16:33:02 [Note] Error reading relay log event: slave SQL thread was killed
180413 16:33:02 [Note] Slave I/O thread killed while connecting to master
180413 16:33:02 [Note] Slave I/O thread exiting, read up to log 'mysql.000004', position 107
180413 16:33:05 [Note] Slave SQL thread initialized, starting replication in log 'mysql.000004' at position 107, relay log '.\DESKTOP-5ECA991-relay-bin.000015' position: 4
180413 16:33:06 [ERROR] Slave I/O: error connecting to master 'root@10.0.0.32:3306' - retry-time: 60  retries: 86400, Error_code: 2003
```
将从库的改为'mysql.000004' at position 107

#### 2
2.错误提示
Slave I/O: Got fatal error 1236 from master when reading data from binary 
log: 'Binary log is not open',Error_code: 1236
原因：没有设置log-bin，在master的my文件中设置log-bin=mysql-bin  //启动二进制日志系统 

### 主从相关单词意义
1. Slave_IO_Running: No是主从同步故障
2. Slave_SQL_Running: Yes是从服务器读取中继日志