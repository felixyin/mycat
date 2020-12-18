# mysql 主从及 mycat

基于 docker 快速跑一个 mysql 主从和 mycat 服务

## 第一次运行需执行

将 mysql 数据存储位置放到宿主机

1. docker-compose up
2. docker cp 6b6dd5efbc23:/var/lib/mysql ./db1
3. docker cp 6b6dd5efbc23:/var/lib/mysql ./db2
4. docker-compose down
5. docker-compose up

## mysql 主从配置

1. 主库配置：

修改 id：server-id = 1
修改：log_bin=master_log

登录 root 执行：

```
create user 'slave' identified by 'slave';
grant all privileges on _._ to slave@'%' #(修改权限时在后面加 with grant option);
```

配置验证：

```
select user,host from user;
show master status;
```

2. 从库配置

修改 id：server-id = 2
注释：#log_bin

/var/lib/mysql/auto.cnf 修改编号在 dbx 中不同即可

stop slave;
登录 master 库并执行： show master status;

切换从库：复制 master status 结果到下面替换，并执行：

```
stop slave;
show slave status;
change master to master_host='db1', master_user='root', master_password='123456', master_port=3306,
master_log_file='master_log.000011';
start slave;
show slave status;
```

> 如果没有错误，则表示主从搭建完毕

## mycat 配置

1. 登录 master mysql，创建 mycat 用户

```
create user 'mycat' identified by 'mycat';
grant all privileges on _._ to mycat@'%' #(修改权限时在后面加 with grant option);
```

2. 配置 config/schema.xml, server.xml

> 分片算法选择够垃圾，问题太多，请建表前谨慎选择。

3. mysql 链接 mycat 测试：

mysql -h127.0.0.1 -uroot -p123456 -P8066 -DTESTDB -A

```
--- SQL 相关脚本
--- 注意大小写

DROP TABLE IF EXISTS T_ADMIN;
DROP TABLE IF EXISTS T_USER;

CREATE TABLE T_USER(
ID INT NOT NULL ,
NAME VARCHAR(32),
PRIMARY KEY (ID)
);

INSERT INTO T_USER(ID, NAME) VALUES(1, 'AAA');
INSERT INTO T_USER(ID, NAME) VALUES(2, 'BBB');
INSERT INTO T_USER(ID, NAME) VALUES(3, 'CCC');
INSERT INTO T_USER(ID, NAME) VALUES(4, 'DDD');
INSERT INTO T_USER(ID, NAME) VALUES(5, 'EEE');
INSERT INTO T_USER(ID, NAME) VALUES(6, 'AAA1');
INSERT INTO T_USER(ID, NAME) VALUES(7, 'BBB1');
INSERT INTO T_USER(ID, NAME) VALUES(8, 'CCC1');
INSERT INTO T_USER(ID, NAME) VALUES(9, 'DDD1');
INSERT INTO T_USER(ID, NAME) VALUES(10, 'EEE2');
INSERT INTO T_USER(ID, NAME) VALUES(11, 'AAA2');
INSERT INTO T_USER(ID, NAME) VALUES(12, 'BBB2');
INSERT INTO T_USER(ID, NAME) VALUES(13, 'CCC2');
INSERT INTO T_USER(ID, NAME) VALUES(14, 'DDD2');
INSERT INTO T_USER(ID, NAME) VALUES(15, 'EEE2');

CREATE TABLE T_ADMIN(
ID INT NOT NULL,
NAME VARCHAR(32),
USER_ID INT,
PRIMARY KEY (ID)
);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(1, 'ADMIN1', 1);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(2, 'ADMIN2', 1);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(3, 'ADMIN3', 2);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(4, 'ADMIN4', 2);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(5, 'ADMIN5', 3);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(6, 'ADMIN6', 3);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(7, 'ADMIN7', 4);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(8, 'ADMIN8', 4);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(9, 'ADMIN9', 5);
INSERT INTO T_ADMIN(ID, NAME, USER_ID) VALUES(10, 'ADMIN0', 5);

-- 查询语法：
SELECT \* FROM T_USER LEFT JOIN T_ADMIN ON T_USER.ID = T_ADMIN.USER_ID;
--查询结果：
```
