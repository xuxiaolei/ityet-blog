---
layout: post
title: Mysql脚本可重复执行语法
categories: mysql
description: Mysql脚本可重复执行语法
index_img: https://xn--6or.us.kg/api/?raw
date: 2018-12-01 09:09:09
tags: [mysql]
---

# 前言

提交相关脚本到生产执行的数据库脚本一般要求可以反复执行，以此记录下相关脚本的重复执行的规范写法。

## 介绍

数据库脚本一般分为：
1.DDL（数据定义语言）操作对象是表，包含：Create、Drop、Alter
2.DML（数据操控语言）操作对象是数据，包含：Insert、Delete、Update
3.DCL（数据控制语言）操作对象是权限、用户，Grant、Revoke

## DDL

#### Create

```sql
DROP TABLE IF EXISTS `USER`;
CREATE TABLE `USER` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `MOBILE` varchar(20) DEFAULT NULL COMMENT '手机',
  `NAME` varchar(20) DEFAULT NULL COMMENT '用户名',
  `PASSWORD` varchar(20) DEFAULT NULL COMMENT '密码',
  `IS_ENABLE` int(2) DEFAULT NULL COMMENT '是否有效 1有效 0无效',
  `CREATE_TIME` datetime DEFAULT NULL,
  `UPDATE_TIME` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `CREATE_BY` varchar(20) NOT NULL DEFAULT 'admin' COMMENT '创建者，记录创建者信息',
  `LAST_UPDATE_BY` varchar(20) NOT NULL DEFAULT 'admin' COMMENT '修改者，记录修改者信息',
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';

```

#### Drop

```sql
DROP TABLE IF EXISTS `USER`;

```

#### Alter

```sql
CREATE PROCEDURE ADD_USER_ADDRESS()
BEGIN
    IF NOT EXISTS (SELECT * FROM information_schema.COLUMNS
							WHERE table_schema = 'test'
							AND table_name = 'USER'
							AND column_name = 'ADDRESS')
	THEN
		ALTER TABLE `USER`
		ADD COLUMN `ADDRESS` varchar(20) NULL COMMENT '用户住址' AFTER `UPDATE_TIME`;
	END IF;
END;
CALL ADD_USER_ADDRESS;
DROP PROCEDURE ADD_USER_ADDRESS;

```

## DML

#### Insert

```sql
DELETE FROM USER WHERE ID = 1;
INSERT INTO `wen`.`USER` (`ID`, `MOBILE`, `NAME`, `PASSWORD`, `IS_DELETED`, `CREATE_TIME`, `LAST_UPDATE_TIME`, `CREATE_BY`, `LAST_UPDATE_BY`)
VALUES ('1', '15888888888', '张三', '123456', '2018-12-01 00:00:00', NULL, 'admin', 'admin');

```

#### Delete

数据库一般不建议也不允许进行物理删除

#### update

```sql
/*备份表与数据*/
DROP TABLE IF EXISTS `USER_20190927`;/*第一次运行保留，后面删除*/
CREATE TABLE USER_20190927 LIKE USER;
INSERT INTO USER_20190927 SELECT * FROM USER;

/*软删除2018-12-01前数据*/
UPDATE USER
SET IS_ENABLE = 0
WHERE IS_ENABLE = 1
AND CREATE_TIME < '2018-12-01 00:00:00'
```

-------------------------

