## 这是一个备份命令

### 作者

陈淼

****

首先，我们为了测试，新建一个测试用的数据库：
```
[gpadmin@gpmagic ~]$ createdb testbackup
```
然后，我们在新创建的数据库中，创建3张表，并插入一些数据，为了后续测试的需要，我们创建Heap表。
```
[gpadmin@gpmagic ~]$ psql testbackup
psql (9.4.24)
Type "help" for help.

testbackup=# CREATE TABLE t1(a int)DISTRIBUTED RANDOMLY;
CREATE TABLE
testbackup=# CREATE TABLE t2(a int)DISTRIBUTED RANDOMLY;
CREATE TABLE
testbackup=# CREATE TABLE t3(a int)DISTRIBUTED RANDOMLY;
CREATE TABLE
testbackup=# INSERT INTO t1 SELECT generate_series(1,10000);
INSERT 0 10000
testbackup=# INSERT INTO t2 SELECT generate_series(1,20000);
INSERT 0 20000
testbackup=# INSERT INTO t3 SELECT generate_series(1,30000);
INSERT 0 30000
```






