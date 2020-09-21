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
下面开始执行第一次备份：
```
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2
20200921:10:45:46.001536-[INFO]-:Start backup process..................................................................
20200921:10:45:46.001536-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2
20200921:10:45:46.001536-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:10:45:46.001536-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
20200921:10:45:46.001536-[INFO]-:Option values: --database testbackup --port 5432 -t  -T
20200921:10:45:46.001536-[INFO]-:Option values: -f  -F  -s  -S
20200921:10:45:46.001536-[INFO]-:Option values: -B 5 --directory /tmp --where  --time-flag 20200921104546
20200921:10:45:46.001536-[INFO]-:Option values: --full  --increment 1 --residue 2 --force-redo
20200921:10:45:46.001536-[INFO]-:Option values: -a  --truncate  --no-error  --encoding UTF8 --compress  --parameter-file
20200921:10:45:46.001536-[NOTICE]-:Try to create backup success log file: /tmp/gpseg-1/db_dumps/20200921104546/backup_done.log
20200921:10:45:46.001536-[INFO]-:Backup directory is /tmp/gpseg-1/db_dumps/20200921104546
20200921:10:45:46.001536-[NOTICE]-:No table gp_toolkit.gp_segment_config or need recreate it
20200921:10:45:46.001536-[INFO]-:Racord in gp_toolkit.gp_segment_config incorrect, reinsert
20200921:10:45:46.001536-[INFO]-:Language plpythonu is OK
20200921:10:45:46.001536-[NOTICE]-:No execute function or need replace, create or replace it
20200921:10:45:46.001536-[NOTICE]-:No backup function or need replace, create or replace it
20200921:10:45:46.001536-[NOTICE]-:No heap stat function or need replace, create or replace it
20200921:10:45:46.001536-[WARN]-:No full backup found before
20200921:10:45:46.001536-[INFO]-:All table number in database is: [3]
20200921:10:45:46.001536-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921104546/backup_done.log
20200921:10:45:46.001536-[INFO]-:Backup all user table in database: testbackup
20200921:10:45:46.001536-[INFO]-:All table should be backup be specified: [3]
20200921:10:45:47.001536-[INFO]-:Try to execute checkpoint success
20200921:10:45:47.001536-[WARN]-:Not found any full backup, will execute full backup this time
20200921:10:45:47.001536-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:10:45:47.001536-[INFO]-:Ignore table number for increment stat not change is: [0]
20200921:10:45:47.001536-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:10:45:48.001536-[INFO]-:Number of tables should be backup is: 3
20200921:10:45:48.001536-[INFO]-:Start backup table testbackup.public.t3
20200921:10:45:48.001536-[INFO]-:Start backup table testbackup.public.t2
20200921:10:45:48.001536-[INFO]-:Start backup table testbackup.public.t1
20200921:10:45:48.001536-[SUCCESS]-:(1/0/3) testbackup.public.t3 SIZE 168894 BYTES TIME 0 S
20200921:10:45:48.001536-[SUCCESS]-:(2/0/3) testbackup.public.t2 SIZE 108894 BYTES TIME 0 S
20200921:10:45:48.001536-[SUCCESS]-:(3/0/3) testbackup.public.t1 SIZE 48894 BYTES TIME 0 S
20200921:10:45:48.001536-[INFO]-:Success execute full backup
20200921:10:45:48.001536-[INFO]-:Finish backup with all success......
```
--increment参数表示，这是一个增量备份，而因为之前没有备份，所以，这一次的增量实际上会是一个全量备份，--directory /tmp表示，我们将备份文件存储到/tmp目录下，--residue 2表示，我们的增量备份将只保留2份增量，也就是说，后续我们一直使用这个参数进行多次增量备份，增量备份的数量不会持续增长。




