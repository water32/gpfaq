通过gpddlrestore命令来进行并行DDL恢复。例如：
```
[gpadmin@smdw bin]$ time ./gpddlrestore --database lotus -f lotus.ddl
20200917:15:38:43.029717-[NOTICE]-:Not specify or out of limit, use default(32): -B
Start to restore ResourceQueue................
ERROR:  resource queue "pg_default" already exists
ALTER QUEUE
Start to restore ResourceQueue................
ERROR:  resource group "admin_group" already exists
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ERROR:  resource group "default_group" already exists
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
ALTER RESOURCE GROUP
Start to restore Role................
ERROR:  role "gpadmin" already exists
Start to restore RoleSetting................
ALTER ROLE
ALTER ROLE
WARNING:  resource group is disabled
HINT:  To enable set gp_resource_manager=group
ALTER ROLE
Start to restore Language................
CREATE LANGUAGE
Start to restore Table................
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
Start to restore ExtTable................
CREATE EXTERNAL TABLE
ALTER EXTERNAL TABLE
Start to restore Index................
ALTER TABLE
ALTER TABLE
ALTER TABLE

real    0m2.578s
user    0m0.977s
sys     0m0.147s
[gpadmin@smdw bin]$
```
这样就恢复好了所有的数据库对象。再进行ddl的重新备份：
```
[gpadmin@smdw bin]$ time ./gpddlbackup --database lotus -f lotus.ddl.new

real    0m1.104s
user    0m0.192s
sys     0m0.198s
```
