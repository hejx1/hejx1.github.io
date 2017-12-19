---
layout: post
title: mysql 使用自增id和uuid作为主键的优劣比较
date: 2017-12-19
tags: mysql    
---

# 测试缘由

一个开发同事做了一个框架，里面主键是uuid，我跟他建议说mysql不要用uuid用自增主键，自增主键效率高，他说不一定高，我说innodb的索引特性导致了自增id做主键是效率最好的，为了拿实际的案例来说服他，所以准备做一个详细的测试。

作为互联网公司，一定有用户表，而且用户表UC_USER基本会有百万记录，所以在这个表基础上准测试数据来进行测试

​         测试过程是目前我想到的多方位的常用的几种类型的sql进行测试，当然可能不太完善，欢迎大家留言提出更加完善的测试方案或者测试sql语句。

### 1、准备表以及数据

UC_USER，自增ID为主键，表结构类似如下：

```
CREATE TABLE `UC_USER` (
  `ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `USER_NAME` varchar(100) DEFAULT NULL COMMENT '用户名',
  `USER_PWD` varchar(200) DEFAULT NULL COMMENT '密码',
  `BIRTHDAY` datetime DEFAULT NULL COMMENT '生日',
  `NAME` varchar(200) DEFAULT NULL COMMENT '姓名',
  `USER_ICON` varchar(500) DEFAULT NULL COMMENT '头像图片',
  `SEX` char(1) DEFAULT NULL COMMENT '性别, 1:男，2:女，3：保密',
  `NICKNAME` varchar(200) DEFAULT NULL COMMENT '昵称',
  `STAT` varchar(10) DEFAULT NULL COMMENT '用户状态，01:正常，02:冻结',
  `USER_MALL` bigint(20) DEFAULT NULL COMMENT '当前所属MALL',
  `LAST_LOGIN_DATE` datetime DEFAULT NULL COMMENT '最后登录时间',
  `LAST_LOGIN_IP` varchar(100) DEFAULT NULL COMMENT '最后登录IP',
  `SRC_OPEN_USER_ID` bigint(20) DEFAULT NULL COMMENT '来源的联合登录',
  `EMAIL` varchar(200) DEFAULT NULL COMMENT '邮箱',
  `MOBILE` varchar(50) DEFAULT NULL COMMENT '手机',
  `IS_DEL` char(1) DEFAULT '0' COMMENT '是否删除',
  `IS_EMAIL_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定邮箱',
  `IS_PHONE_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定手机',
  `CREATER` bigint(20) DEFAULT NULL COMMENT '创建人',
  `CREATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',
  `UPDATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '修改日期',
  `PWD_INTENSITY` char(1) DEFAULT NULL COMMENT '密码强度',
  `MOBILE_TGC` char(64) DEFAULT NULL COMMENT '手机登录标识',
  `MAC` char(64) DEFAULT NULL COMMENT 'mac地址',
  `SOURCE` char(1) DEFAULT '0' COMMENT '1:WEB,2:IOS,3:ANDROID,4:WIFI,5:管理系统, 0:未知',
  `ACTIVATE` char(1) DEFAULT '1' COMMENT '激活，1：激活，0：未激活',
  `ACTIVATE_TYPE` char(1) DEFAULT '0' COMMENT '激活类型，0：自动，1：手动',
  PRIMARY KEY (`ID`),
  UNIQUE KEY `USER_NAME` (`USER_NAME`),
  KEY `MOBILE` (`MOBILE`),
  KEY `IDX_MOBILE_TGC` (`MOBILE_TGC`,`ID`),
  KEY `IDX_EMAIL` (`EMAIL`,`ID`),
  KEY `IDX_CREATE_DATE` (`CREATE_DATE`,`ID`),
  KEY `IDX_UPDATE_DATE` (`UPDATE_DATE`)
) ENGINE=InnoDB AUTO_INCREMENT=7122681 DEFAULT CHARSET=utf8 COMMENT='用户表'
```

### 2、500w数据测试

- 录入500w数据,自增id节省一半磁盘空间

确定两个表数据量

```
# 自增id为主键的表
mysql> select count(1) from UC_USER;
+----------+
| count(1) |
+----------+
|  5720112 |
+----------+
1 row in set (0.00 sec)
 
mysql>
 
# uuid为主键的表
mysql> select count(1) from UC_USER_PK_VARCHAR_1;
+----------+
| count(1) |
+----------+
|  5720112 |
+----------+
1 row in set (1.91 sec)
```

占据的空间容量来看，自增ID比UUID小一半左右。

| **主键类型** | **数据文件大小**                               | **占据容量** |
| -------- | ---------------------------------------- | -------- |
| 自增ID     | -rw-rw---- 1 mysql mysql 2.5G Aug 11 18:29 UC_USER.ibd | 2.5 G    |
| UUID     | -rw-rw---- 1 mysql mysql 5.4G Aug 15 15:11 UC_USER_PK_VARCHAR_1.ibd | 5.4 G    |

- 单个数据走索引查询，自增id和uuid相差不大

| **主键类型** | **SQL语句**                                | **执行时间 (秒)** |
| -------- | ---------------------------------------- | ------------ |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER t WHERE t.MOBILE ='14782121512'; | 0.118        |
|          |                                          |              |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER_PK_VARCHAR_1  t WHERE t.MOBILE ='14782121512'; | 0.117        |
|          |                                          |              |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER t WHERE t.MOBILE IN( '14782121512','13761460105'); | 0.049        |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER_PK_VARCHAR_1 t WHERE t.MOBILE IN('14782121512','13761460105'); | 0.040        |
|          |                                          |              |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER t WHERE t.CREATE_DATE='2013-11-24 10:26:36' ; | 0.139        |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.UC_USER_PK_VARCHAR_1 t WHERE t.CREATE_DATE='2013-11-24 10:26:43' ; | 0.126        |

- 范围like查询，自增ID性能优于UUID

| **主键类型**                       | **SQL语句**                                | **执行时间 (秒)** |
| ------------------------------ | ---------------------------------------- | ------------ |
| （1）模糊范围查询1000条数据，自增ID性能要好于UUID |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER` t WHERE t.`MOBILE` LIKE '147%' LIMIT 1000; | 1.784        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`MOBILE` LIKE '147%' LIMIT 1000; | 3.196        |
| （2）日期范围查询20条数据，自增ID稍微弱于UUID    |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER` t WHERE t.`CREATE_DATE` > '2016-08-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 20; | 0.601        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-08-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 20; | 0.543        |
| （3）范围查询200条数据，自增ID性能要好于UUID    |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 200; | 2.314        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 200; | 3.229        |
| 范围查询总数量，自增ID要好于UUID            |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE COUNT(1) FROM test.`UC_USER` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36'  ; | 0.514        |
| UUID                           | SELECT SQL_NO_CACHE COUNT(1) FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36'  ; | 1.092        |

**PS：在有缓存的情况下，两者执行效率相差很小。**

- 写入测试，自增ID是UUID的4倍

| **主键类型** | **SQL语句**                                | **执行时间 (秒)** |
| -------- | ---------------------------------------- | ------------ |
| 自增ID     | UPDATE test.`UC_USER` t SET t.`MOBILE_TGC`='T2' WHERE t.`CREATE_DATE` > '2016-05-03 10:26:36' AND t.`CREATE_DATE` <'2016-05-04 00:00:00'  ; | 1.419        |
| UUID     | UPDATE test.`UC_USER_PK_VARCHAR_1` t SET t.`MOBILE_TGC`='T2' WHERE t.`CREATE_DATE` > '2016-05-03 10:26:36' AND t.`CREATE_DATE` <'2016-05-04 00:00:00'  ; | 5.639        |
|          |                                          |              |
| 自增ID     | INSERT INTO test.`UC_USER`(   ID,   `USER_NAME`,   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`,   `MOBILE`,   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` ) SELECT       NULL,    CONCAT('110',`USER_NAME`,8),   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`, CONCAT('110',TRIM(`MOBILE`)),   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` FROM `test`.`UC_USER_1` LIMIT 100; | 0.105        |
| UUID     | INSERT INTO test.`UC_USER_PK_VARCHAR_1`(    ID,   `USER_NAME`,   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`,   `MOBILE`,   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` ) SELECT         UUID(),   CONCAT('110',`USER_NAME`,8),   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`, CONCAT('110',TRIM(`MOBILE`)),   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` FROM `test`.`UC_USER_1` LIMIT 100; | 0.424        |

- 备份和恢复，自增ID性能优于UUID

| **主键类型**         | **SQL语句**                                | **执行时间 (秒)** |
| ---------------- | ---------------------------------------- | ------------ |
| **Mysql dump备份** |                                          |              |
| 自增ID             | time mysqldump -utim -ptimgood -h192.168.121.63 test UC_USER_500> UC_USER_500.sql | 28.59秒       |
| UUID             | time mysqldump -utim -ptimgood -h192.168.121.63 test UC_USER_PK_VARCHAR_500> UC_USER_PK_VARCHAR_500.sql | 31.08秒       |
| **MySQL恢复**      |                                          |              |
| 自增ID             | time mysql  -utim -ptimgood -h192.168.121.63  test < UC_USER_500.sql | 7m36.601s    |
| UUID             | time mysql  -utim -ptimgood -h192.168.121.63  test < UC_USER_PK_VARCHAR_500.sql | 9m42.472s    |
|                  |                                          |              |

### 3、500W总结

**在500W记录表的测试下：**

1. 普通单条或者20条左右的记录检索，uuid为主键的相差不大几乎效率相同；
2. 但是范围查询特别是上百成千条的记录查询，自增id的效率要大于uuid；
3. 在范围查询做统计汇总的时候，自增id的效率要大于uuid；
4. 在存储上面，自增id所占的存储空间是uuid的1/2；
5. 在备份恢复上，自增ID主键稍微优于UUID。

### 4、1000W数据测试

- 录入1000W条数据记录，查看存储空间

```
# 自增id为主键的表
mysql> use test;
Database changed
mysql> select count(1) from UC_USER_1;
+----------+
| count(1) |
+----------+
| 10698102 |
+----------+
1 row in set (27.42 sec)
 
# uuid为主键的表
mysql> select count(1) from UC_USER_PK_VARCHAR_1;
+----------+
| count(1) |
+----------+
| 10698102 |
+----------+
1 row in set (0.00 sec)
 
mysql>
```

**占据的空间容量来看，自增ID比UUID小一半左右：**

| **主键类型** | **数据文件大小**                               | **占据容量  ** |
| -------- | ---------------------------------------- | ---------- |
| 自增ID     | -rw-rw---- 1 mysql mysql 4.2G Aug 20 23:08 UC_USER_1.ibd | 4.2 G      |
| UUID     | -rw-rw---- 1 mysql mysql 8.8G Aug 20 18:20 UC_USER_PK_VARCHAR_1.ibd | 8.8 G      |

- 单个数据走索引查询，自增id和 uuid效率比是：(2~3):1

| **主键类型** | **SQL语句**                                | **执行时间 (秒)** |
| -------- | ---------------------------------------- | ------------ |
| 单条记录查询   |                                          |              |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_1` t WHERE t.`MOBILE` ='14782121512'; | 0.069        |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`MOBILE` ='14782121512'; | 0.274        |
| 小范围查询    |                                          |              |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_1` t WHERE t.`MOBILE` IN( '14782121512','13761460105'); | 0.050        |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`MOBILE` IN('14782121512','13761460105'); | 0.151        |
| 根据日期查询   |                                          |              |
| 自增ID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_1` t WHERE t.`CREATE_DATE`='2013-11-24 10:26:36' ; | 0.269        |
| UUID     | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE`='2013-11-24 10:26:43' ; | 0.810        |

- 范围like查询，自增ID性能优于UUID，比值(1.5~2)：1

| **主键类型**                       | **SQL语句**                                | **执行时间 (秒)** |
| ------------------------------ | ---------------------------------------- | ------------ |
| （1）模糊范围查询1000条数据，自增ID性能要好于UUID |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER` t WHERE t.`MOBILE` LIKE '147%' LIMIT 1000; | 2.398        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`MOBILE` LIKE '147%' LIMIT 1000; | 5.872        |
| （2）日期范围查询20条数据，自增ID稍微弱于UUID    |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_1` t WHERE t.`CREATE_DATE` > '2016-08-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 20; | 0.765        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-08-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 20; | 1.090        |
| （3）范围查询200条数据，自增ID性能要好于UUID    |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 200; | 1.569        |
| UUID                           | SELECT SQL_NO_CACHE t.* FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36' ORDER BY t.`UPDATE_DATE` DESC LIMIT 200; | 2.597        |
| 范围查询总数量，自增ID要好于UUID            |                                          |              |
| 自增ID                           | SELECT SQL_NO_CACHE COUNT(1) FROM test.`UC_USER_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36'  ; | 1.129        |
| UUID                           | SELECT SQL_NO_CACHE COUNT(1) FROM test.`UC_USER_PK_VARCHAR_1` t WHERE t.`CREATE_DATE` > '2016-07-01 10:26:36'  ; | 2.302        |

- 写入测试，自增ID比UUID效率高，比值(3~10)：1

| **主键类型**    | **SQL语句**                                | **执行时间 (秒)** |
| ----------- | ---------------------------------------- | ------------ |
| **修改一天的记录** |                                          |              |
| 自增ID        | UPDATE test.`UC_USER_1` t SET t.`MOBILE_TGC`='T2' WHERE t.`CREATE_DATE` > '2016-05-03 10:26:36' AND t.`CREATE_DATE` <'2016-05-04 00:00:00'  ; | 2.685        |
| UUID        | UPDATE test.`UC_USER_PK_VARCHAR_1` t SET t.`MOBILE_TGC`='T2' WHERE t.`CREATE_DATE` > '2016-05-03 10:26:36' AND t.`CREATE_DATE` <'2016-05-04 00:00:00'  ; | 26.521       |
| 录入数据        |                                          |              |
| 自增ID        | INSERT INTO test.`UC_USER_1`(   ID,   `USER_NAME`,   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`,   `MOBILE`,   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` ) SELECT       NULL,    CONCAT('110',`USER_NAME`,8),   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`, CONCAT('110',TRIM(`MOBILE`)),   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` FROM `test`.`UC_USER_1` LIMIT 100; | 0.534        |
| UUID        | INSERT INTO test.`UC_USER_PK_VARCHAR_1`(    ID,   `USER_NAME`,   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`,   `MOBILE`,   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` ) SELECT         UUID(),   CONCAT('110',`USER_NAME`,8),   `USER_PWD`,   `BIRTHDAY`,   `NAME`,   `USER_ICON`,   `SEX`,   `NICKNAME`,   `STAT`,   `USER_MALL`,   `LAST_LOGIN_DATE`,   `LAST_LOGIN_IP`,   `SRC_OPEN_USER_ID`,   `EMAIL`, CONCAT('110',TRIM(`MOBILE`)),   `IS_DEL`,   `IS_EMAIL_CONFIRMED`,   `IS_PHONE_CONFIRMED`,   `CREATER`,   `CREATE_DATE`,   `UPDATE_DATE`,   `PWD_INTENSITY`,   `MOBILE_TGC`,   `MAC`,   `SOURCE`,   `ACTIVATE`,   `ACTIVATE_TYPE` FROM `test`.`UC_USER_1` LIMIT 100; | 1.716        |

- 备份和恢复，自增ID性能优于UUID

| **主键类型**         | **SQL语句**                                | **执行时间 (秒)** |
| ---------------- | ---------------------------------------- | ------------ |
| **Mysql dump备份** |                                          |              |
| 自增ID             | time mysqldump -utim -ptimgood -h192.168.121.63 test UC_USER_1> UC_USER_1.sql | 0m50.548s    |
| UUID             | time mysqldump -utim -ptimgood -h192.168.121.63 test UC_USER_PK_VARCHAR_1> UC_USER_PK_VARCHAR_1.sql | 0m58.590s    |
| **MySQL恢复**      |                                          |              |
| 自增ID             | time mysql -utim -ptimgood -h192.168.121.63 test < UC_USER_1.sql | 17m30.822s   |
| UUID             | time mysql -utim -ptimgood -h192.168.121.63 test < UC_USER_PK_VARCHAR_1.sql | 23m6.360s    |
|                  |                                          |              |

### 5、1000W总结

**在1000W记录表的测试下：**

1. 普通单条或者20条左右的记录检索，自增主键效率是uuid主键的2到3倍；
2. 但是范围查询特别是上百成千条的记录查询，自增id的效率要大于uuid；
3. 在范围查询做统计汇总的时候，自增id主键的效率是uuid主键1.5到2倍；
4. 在存储上面，自增id所占的存储空间是uuid的1/2；
5. 在写入上面，自增ID主键的效率是UUID主键的3到10倍，相差比较明显，特别是update小范围之内的数据上面。
6. 在备份恢复上，自增ID主键稍微优于UUID。

### 6、MySQL分布式架构的取舍

分布式架构，意味着需要多个实例中保持一个表的主键的唯一性。这个时候普通的单表自增ID主键就不太合适，因为多个mysql实例上会遇到主键全局唯一性问题。

### 6.1、自增ID主键+步长，适合中等规模的分布式场景

在每个集群节点组的master上面，设置（auto_increment_increment），让目前每个集群的起始点错开 1，步长选择大于将来基本不可能达到的切分集群数，达到将 ID 相对分段的效果来满足全局唯一的效果。



**<font color='red'>优点是：实现简单，后期维护简单，对应用透明。</font>**

**<font color='red'>缺点是：第一次设置相对较为复杂，因为要针对未来业务的发展而计算好足够的步长;</font>**





**规划：**

比如计划总共N个节点组，那么第i个节点组的my.cnf的配置为：

auto_increment_offset  i

auto_increment_increment  N

 

假如规划48个节点组，N为48，现在配置第8个节点组，这个i为8，第8个节点组的my.cnf里面的配置为：

auto_increment_offset  8

auto_increment_increment  48

### 6.2、UUID，适合小规模的分布式环境

对于InnoDB这种聚集主键类型的引擎来说，数据会按照主键进行排序，由于UUID的无序性，InnoDB会产生巨大的IO压力，而且由于索引和数据存储在一起，字符串做主键会造成存储空间增大一倍。

 

在存储和检索的时候，innodb会对主键进行物理排序，这对auto_increment_int是个好消息，因为后一次插入的主键位置总是在最后。但是对uuid来说，这却是个坏消息，因为uuid是杂乱无章的，每次插入的主键位置是不确定的，可能在开头，也可能在中间，在进行主键物理排序的时候，势必会造成大量的 IO操作影响效率，在数据量不停增长的时候，特别是数据量上了千万记录的时候，读写性能下降的非常厉害。

 

**<font color='red'>优点：搭建比较简单，不需要为主键唯一性的处理。</font>**

**<font color='red'>缺点：占用两倍的存储空间（在云上光存储一块就要多花2倍的钱），后期读写性能下降厉害。</font>**

### 6.3、雪花算法自造全局自增ID，适合大数据环境的分布式场景

由twitter公布的开源的分布式id算法snowflake（Java版本）

**IdWorker.java:**

```
package com.demo.elk;
import org.slf4j.Logger; 
import org.slf4j.LoggerFactory;
 
public class IdWorker {
   
    protected static final Logger LOG = LoggerFactory.getLogger(IdWorker.class);
    
    private long workerId;
    private long datacenterId;
    private long sequence = 0L;
 
    private long twepoch = 1288834974657L;
 
    private long workerIdBits = 5L;
    private long datacenterIdBits = 5L;
    private long maxWorkerId = -1L ^ (-1L << workerIdBits);
    private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
    private long sequenceBits = 12L;
 
    private long workerIdShift = sequenceBits;
    private long datacenterIdShift = sequenceBits + workerIdBits;
    private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
    private long sequenceMask = -1L ^ (-1L << sequenceBits);
 
    private long lastTimestamp = -1L;
 
    public IdWorker(long workerId, long datacenterId) {
        // sanity check for workerId
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
        LOG.info(String.format("worker starting. timestamp left shift %d, datacenter id bits %d, worker id bits %d, sequence bits %d, workerid %d", timestampLeftShift, datacenterIdBits, workerIdBits, sequenceBits, workerId));
    }
 
    public synchronized long nextId() {
        long timestamp = timeGen();
 
        if (timestamp < lastTimestamp) {
            LOG.error(String.format("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp));
            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }
 
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            if (sequence == 0) {
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0L;
        }
 
        lastTimestamp = timestamp;
 
        return ((timestamp - twepoch) << timestampLeftShift) | (datacenterId << datacenterIdShift) | (workerId << workerIdShift) | sequence;
    }
 
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }
 
    protected long timeGen() {
        return System.currentTimeMillis();
    }
}
```

**测试生成ID的测试类，IdWorkerTest.java：**

```
package com.demo.elk;
 
import java.util.HashSet;
import java.util.Set;
 
public class IdWorkerTest {
          
    static class IdWorkThread implements Runnable {
        private Set<Long> set;
        private IdWorker idWorker;
 
        public IdWorkThread(Set<Long> set, IdWorker idWorker) {
            this.set = set;
            this.idWorker = idWorker;
        }
 
        public void run() {
            while (true) {
                long id = idWorker.nextId();
                System.out.println("            real id:" + id);
                if (!set.add(id)) {
                    System.out.println("duplicate:" + id);
                }
            }
        }
    }
 
    public static void main(String[] args) {
        Set<Long> set = new HashSet<Long>();
        final IdWorker idWorker1 = new IdWorker(0, 0);
        final IdWorker idWorker2 = new IdWorker(1, 0);
        Thread t1 = new Thread(new IdWorkThread(set, idWorker1));
        Thread t2 = new Thread(new IdWorkThread(set, idWorker2));
        t1.setDaemon(true);
        t2.setDaemon(true);
        t1.start();
        t2.start();
        try {
            Thread.sleep(30000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 7、总结

**（1）单实例或者单节点组：**

经过500W、1000W的单机表测试，自增ID相对UUID来说，自增ID主键性能高于UUID，磁盘存储费用比UUID节省一半的钱。所以在单实例上或者单节点组上，使用自增ID作为首选主键。

 

**（2）分布式架构场景：**

- 20个节点组下的小型规模的分布式场景，为了快速实现部署，可以采用多花存储费用、牺牲部分性能而使用UUID主键快速部署；
- 20到200个节点组的中等规模的分布式场景，可以采用自增ID+步长的较快速方案。
- 200以上节点组的大数据下的分布式场景，可以借鉴类似twitter雪花算法构造的全局自增ID作为主键。

原文链接：<http://blog.csdn.net/mchdba/article/details/52336203>