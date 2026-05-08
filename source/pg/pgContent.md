---
title: pg
date: 2024-05-11 17:04:17
tags: pg
description: pg
---

### 创建pg数据库

```shell
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql14-server
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14

sudo -u postgres psql
--设置postgres的用户密码
ALTER USER postgres WITH PASSWORD  '123456';
--创建用户
CREATE USER dong_pg WITH PASSWORD '123456';
--赋予用户创建数据库的权限  
ALTER ROLE dong_pg CREATEDB;
-- 赋予用户创建角色的权限  
ALTER ROLE dong_pg CREATEROLE;  
-- 赋予用户超级用户权限（谨慎使用）
ALTER ROLE dong_pg SUPERUSER;  
--使用dong_pg用户进入dong_pg数据库
psql -U dong_pg -h localhost -p 5432 -d dong_pg

psql -U postgres -h localhost -p 5432

systemctl list-units|grep postgresql
要在win11使用Navicat连接不上数据库操作：
在root用户下执行：
firewall-cmd --list-ports --查看启的端口
firewall-cmd --query-port=5432/tcp --查询5432端口有没有启
firewall-cmd --zone=public --add-port=5432/tcp --permanent --添加5432端口 permanent表示永久启用
firewall-cmd --reload  --重新加载防火墙
systemctl status postgresql-14 --查看数据库状态
systemctl restart postgresql-14 --重启数据库
在 /var/lib/pgsql/14/data/postgresql.conf 中添加：
listen_addresses = '*'
在 /var/lib/pgsql/14/data/pg_hba.conf 中添加：
host    all             all             0.0.0.0/0               md5
```

### 给创建的数据库赋予权限

```postgresql
--管理员进入psql
sudo -u postgres psql
--命令行创建用户和数据库
--参数库
create user mdb with password '123456';
create database dmdb owner mdb;
--交易库
create user tradedb with password '123456';
create database dtradedb owner tradedb;
--批量库
create user pbatdb with password '123456';
create database dpbatdb owner pbatdb;
--授权
ALTER DATABASE dmdb OWNER TO mdb;
--切换到数据库
\c dmdb
-- 1. 允许用户连接到数据库
GRANT CONNECT ON DATABASE dmdb TO mdb;

-- 2. 允许用户在 public 模式下创建新的对象（如创建表、索引）
GRANT CREATE ON SCHEMA public TO mdb;

-- 3. 允许用户使用 public 模式（必须有 USAGE 才能访问其中的对象）
GRANT USAGE ON SCHEMA public TO mdb;

-- 4. 授予对 public 模式下“未来”创建的所有表的 增删改查 权限
-- 这样以后你创建新表，就不需要每次都重新授权了
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES ON TABLES TO mdb;

-- 5. (可选) 如果你已经创建了一些表，授予对现有所有表的权限
GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE ON ALL TABLES IN SCHEMA public TO mdb;

-- 6. (重要) 授予对序列的权限（如果表中有自增 ID，必须授权，否则插入会报错）
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO mdb;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO mdb;


```

```postgresql
创建数据库用户流程：
create user tradedb1 with password '123456';
create database dtradedb1 owner tradedb1;

GRANT CONNECT ON DATABASE dtradedb1 TO tradedb1;

GRANT CREATE ON SCHEMA public TO tradedb1;

GRANT USAGE ON SCHEMA public TO tradedb1;

ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES ON TABLES TO tradedb1;

GRANT SELECT, INSERT, UPDATE, DELETE, TRUNCATE ON ALL TABLES IN SCHEMA public TO tradedb1;

GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO tradedb1;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO tradedb1;
```



### 查询数据库最大连接数

```postgresql
SELECT 
    name, 
    setting AS "最大连接数", 
    (SELECT count(*) FROM pg_stat_activity) AS "当前活跃连接数"
FROM pg_settings 
WHERE name = 'max_connections';
```

### pg查询锁表

```sql
SELECT 
    n.nspname AS schema_name,
    c.relname AS table_name,
    l.locktype,
    l.mode AS lock_mode,
    l.GRANTED,
    a.pid,
    a.usename,
    age(now(), a.query_start) AS query_duration,
    a.query
FROM pg_locks l
JOIN pg_class c ON l.relation = c.oid
JOIN pg_namespace n ON c.relnamespace = n.oid
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE 
    l.locktype = 'relation'  -- 仅表级锁
    AND n.nspname NOT IN ('pg_catalog', 'information_schema')
    AND a.datname = current_database()
ORDER BY query_duration DESC;

--解锁
SELECT pg_terminate_backend(pid);
```



### 重启数据库

```shell
--重启
sudo systemctl restart postgresql-14
--查看状态
sudo systemctl status postgresql-14
```



### pg数据库信息

```
mdb = postgres://mdb:123456@192.168.171.128:5432/dmdb
tradedb = postgres://tradedb:123456@192.168.171.128:5432/dtradedb
pbatdb = postgres://pbatdb:123456@192.168.171.128:5432/dpbatdb
```



### 查询分区表的内容

```sql
SELECT
    p.relname AS partition_name,
    pg_get_expr(p.relpartbound, 0) AS partition_expression
FROM
    pg_class p
JOIN
    pg_inherits i ON p.oid = i.inhrelid
WHERE
    i.inhparent = 'parent_table_name'::regclass;
```



### pg AT操作多个数据库并提交

```c
	if(pgconnect(MDB) < 0) {
        LOG_ERROR("Connect to database failed");
        return -1;
    }

    if(pgconnect(TRADEDB) < 0) {
        LOG_ERROR("Connect to tradedb database failed");
        return -1;
    }
	/*dmdb来源于 exec sql connect to :target as :conn_name user :user identified by :passwd; 中的conn_name*/
    EXEC sql at dmdb select cur_date, last_date into :cur_date, :last_date from fd_busi_date where busi_type ='04';
    EXEC SQL AT dmdb savepoint update_mdb_save;
    EXEC sql at dmdb update fd_busi_date set cur_date = :update_date, last_date = :last_date, last_mod_stamp = :update_time_stamp where busi_type ='04';
    EXEC SQL AT dtradedb savepoint update_dtradedb_save;
    get_sys_time_stamp(update_time_stamp);
    exec sql at dtradedb update sd_account_note set timelimit =3, last_mod_stamp = :update_time_stamp
    where customerid ='44011000002205' and 
    savingbondacct ='020444010001235192' and 
    kindcode ='211701';
    exec sql at dmdb commit;
    exec sql at dtradedb commit;
```

### pg中2PC操作

```sql
--查询没有完成提交的任务id
-- 查询所有未完成的 2PC 事务（推荐）
SELECT 
    gid AS transaction_id,      -- 全局事务ID（即你指定的标识符）
    prepared,                   -- 进入 prepared 状态的时间
    owner,                      -- 事务所有者
    database,                   -- 所属数据库
    transaction AS xid          -- 内部事务ID（XID）
FROM pg_prepared_xacts
ORDER BY prepared;

--提交（确认业务逻辑已完成）
COMMIT PREPARED 'your_gid';

-- 或回滚（确认需废弃）需要在对应的数据库操作回滚
ROLLBACK PREPARED 'your_gid';
```



### pg FDW(Foreign Data Wrapper, 外部数据包装器)

```postgresql
--1.开启扩展
CREATE EXTENSION postgres_fdw;
--2.定义远程服务器
CREATE SERVER foreign_trade_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '...', dbname 'dtradedb');
--3.定义用户映射
CREATE USER MAPPING FOR current_user SERVER foreign_trade_server OPTIONS (user '...', password '...');
--4.创建外部表
CREATE FOREIGN TABLE remote_sd_account_note (...) SERVER foreign_trade_server OPTIONS (table_name 'sd_account_note');


--以mdb为例
CREATE EXTENSION IF NOT EXISTS pg_mdb_fdw;
错误:  无法打开扩展控制文件 "/usr/pgsql-14/share/extension/pg_mdb_fdw.control": 没有那个文件或目录
需要安装 contrib 模块
yum install postgresql14-contrib
--查询数据库有没有postgres_fdw
SELECT name, default_version, comment 
FROM pg_available_extensions 
WHERE name LIKE 'post%';


--dmdb进入postgres超级用户
sudo -u postgres /usr/pgsql-14/bin/psql -d dmdb
--正式创建 FDW
-- 1. 准备工作  postgres_fdw 是标准的,不能更改
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- 2. 定义另外两个服务器 (ALTER SERVER srv_trade RENAME TO srv_tradedb;)
CREATE SERVER srv_tradedb FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '192.168.171.128', dbname 'dtradedb');
GRANT USAGE ON FOREIGN SERVER srv_tradedb TO mdb;  --授予外部服务器的权限
CREATE SERVER srv_pbatdb  FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '192.168.171.128', dbname 'dpbatdb');
GRANT USAGE ON FOREIGN SERVER srv_pbatdb TO mdb;   --授予外部服务器的权限

-- 3. 用户映射（假设账号一致）
CREATE USER MAPPING FOR postgres SERVER srv_tradedb OPTIONS (user 'tradedb', password '123456'); --postgres权限
CREATE USER MAPPING FOR mdb SERVER srv_tradedb OPTIONS (user 'tradedb', password '123456');

CREATE USER MAPPING FOR postgres SERVER srv_pbatdb  OPTIONS (user 'pbatdb', password '123456');  --postgres权限
CREATE USER MAPPING FOR mdb SERVER srv_pbatdb  OPTIONS (user 'pbatdb', password '123456');

--3.1删除用户映射
DROP USER MAPPING IF EXISTS FOR postgres SERVER srv_pbatdb;
DROP USER MAPPING IF EXISTS FOR mdb SERVER srv_pbatdb;

-- 4. 重点：创建独立的命名空间进行隔离
CREATE SCHEMA trade_link;
GRANT USAGE ON SCHEMA trade_link TO mdb;
CREATE SCHEMA pbat_link;
GRANT USAGE ON SCHEMA pbat_link TO mdb;

-- 5. 批量导入：即使对方有上万张表，这两行也瞬间完成  (GRANT USAGE ON FOREIGN SERVER srv_tradedb TO PUBLIC;)
IMPORT FOREIGN SCHEMA public FROM SERVER srv_tradedb INTO trade_link;
IMPORT FOREIGN SCHEMA public FROM SERVER srv_pbatdb  INTO pbat_link;

--6.授予权限
授予 Schema 的所有权限
让 mdb 能够在 trade_link 这个命名空间里“横着走”：
-- 允许 mdb 查看和在其中创建对象
GRANT ALL PRIVILEGES ON SCHEMA trade_link TO mdb;
GRANT ALL PRIVILEGES ON SCHEMA pbat_link TO mdb;
--将 trade_link 下所有表的所有权限（SELECT, INSERT, UPDATE, DELETE 等）给 mdb
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA trade_link TO mdb; --（访问外部表sd_account_note的权限不足）
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA pbat_link TO mdb;
-- 针对未来可能新导入或创建的表，也自动授权（可选）
ALTER DEFAULT PRIVILEGES IN SCHEMA trade_link GRANT ALL ON TABLES TO mdb;
ALTER DEFAULT PRIVILEGES IN SCHEMA pbat_link GRANT ALL ON TABLES TO mdb;

--怎么查看
查服务器：\des
查用户映射：\deu
查外部表：\det trade_link.*
```

