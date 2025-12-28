---
title: pg
date: 2024-05-11 17:04:17
tags: pg
description: pg
---

### 1.创建pg数据库

```
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



### 2.查询分区表的内容

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

