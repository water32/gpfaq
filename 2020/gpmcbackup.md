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
下面的测试，为了节省篇幅，我们将节省不必要的输出信息。

下面开始执行第一次备份：
```
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:04:10.003846-[INFO]-:Start backup process..................................................................
20200921:12:04:10.003846-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:04:10.003846-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:04:10.003846-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:04:10.003846-[NOTICE]-:Try to create backup success log file: /tmp/gpseg-1/db_dumps/20200921120410/backup_done.log
20200921:12:04:10.003846-[INFO]-:Backup directory is /tmp/gpseg-1/db_dumps/20200921120410
。。。。。。
20200921:12:04:11.003846-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:04:11.003846-[INFO]-:Ignore table number for increment stat not change is: [0]
20200921:12:04:11.003846-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:04:12.003846-[INFO]-:Number of tables should be backup is: 3
20200921:12:04:12.003846-[INFO]-:Start backup table testbackup.public.t3
20200921:12:04:12.003846-[INFO]-:Start backup table testbackup.public.t2
20200921:12:04:12.003846-[INFO]-:Start backup table testbackup.public.t1
20200921:12:04:12.003846-[SUCCESS]-:(1/0/3) testbackup.public.t3 SIZE 71521 BYTES TIME 0 S
20200921:12:04:12.003846-[SUCCESS]-:(2/0/3) testbackup.public.t2 SIZE 46258 BYTES TIME 0 S
20200921:12:04:12.003846-[SUCCESS]-:(3/0/3) testbackup.public.t1 SIZE 21009 BYTES TIME 0 S
20200921:12:04:12.003846-[INFO]-:Success execute full backup
20200921:12:04:12.003846-[INFO]-:Finish backup with all success......
```
--increment参数表示，这是一个增量备份，而因为之前没有备份，所以，这一次的增量实际上会是一个全量备份，--directory /tmp表示，我们将备份文件存储到/tmp目录下，--residue 2表示，我们的增量备份将只保留2份增量，也就是说，后续我们一直使用这个参数进行多次增量备份，增量备份的数量不会持续增长。--compress zstd表示，备份出的数据文件采用zstd压缩。

在没有对数据进行任何修改的情况下，我们再次进行备份操作，因为我们的命令是可以识别Heap表是否发生了变化，所以，预期的情况是，这一次增量备份将不会有任何表被备份了。
```
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:09:42.004362-[INFO]-:Start backup process..................................................................
20200921:12:09:42.004362-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:09:42.004362-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:09:42.004362-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:09:42.004362-[NOTICE]-:Try to create backup success log file: /tmp/gpseg-1/db_dumps/20200921120942/backup_done.log
20200921:12:09:42.004362-[INFO]-:Backup directory is /tmp/gpseg-1/db_dumps/20200921120942
。。。。。。
20200921:12:09:42.004362-[WARN]-:No backup need merge
20200921:12:09:42.004362-[INFO]-:All table number in database is: [3]
20200921:12:09:42.004362-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921120942/backup_done.log
20200921:12:09:42.004362-[INFO]-:Backup all user table in database: testbackup
20200921:12:09:42.004362-[INFO]-:All table should be backup be specified: [3]
20200921:12:09:42.004362-[INFO]-:Try to execute checkpoint success
20200921:12:09:42.004362-[INFO]-:Last full backup time is: [20200921120410]
20200921:12:09:43.004362-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:09:43.004362-[INFO]-:Ignore table number for increment stat not change is: [3]
20200921:12:09:43.004362-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:09:44.004362-[INFO]-:No table will be backup, exit
```
接下来，我们尝试修改一点数据，之后再尝试增量备份：
```
[gpadmin@gpmagic ~]$ psql testbackup
psql (9.4.24)
Type "help" for help.

testbackup=# DELETE FROM t3 WHERE random() < 0.001;
DELETE 27
testbackup=# \q
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:10:14.004705-[INFO]-:Start backup process..................................................................
20200921:12:10:14.004705-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:10:14.004705-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:10:14.004705-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:10:14.004705-[NOTICE]-:Try to create backup success log file: /tmp/gpseg-1/db_dumps/20200921121014/backup_done.log
20200921:12:10:14.004705-[INFO]-:Backup directory is /tmp/gpseg-1/db_dumps/20200921121014
。。。。。。
20200921:12:10:14.004705-[WARN]-:20200921120942 : Is an invalid backup path and a garbage backup path
20200921:12:10:14.004705-[WARN]-:No backup need merge
20200921:12:10:14.004705-[INFO]-:All table number in database is: [3]
20200921:12:10:14.004705-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921121014/backup_done.log
20200921:12:10:14.004705-[INFO]-:Backup all user table in database: testbackup
20200921:12:10:14.004705-[INFO]-:All table should be backup be specified: [3]
20200921:12:10:14.004705-[INFO]-:Try to execute checkpoint success
20200921:12:10:14.004705-[INFO]-:Last full backup time is: [20200921120410]
20200921:12:10:14.004705-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:10:14.004705-[INFO]-:Ignore table number for increment stat not change is: [2]
20200921:12:10:14.004705-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:10:15.004705-[INFO]-:Number of tables should be backup is: 1
20200921:12:10:15.004705-[INFO]-:Start backup table testbackup.public.t3
20200921:12:10:15.004705-[SUCCESS]-:(1/0/1) testbackup.public.t3 SIZE 71459 BYTES TIME 0 S
20200921:12:10:15.004705-[INFO]-:Finish backup with all success......
```
我们从输出中可以看出，刚刚被修改的表，被重新备份了，而且，上一次没有任何数据被备份的增量备份，被识别为无效备份。我们看一下备份目录下的情况：
```
[gpadmin@gpmagic ~]$ ll /tmp/gpseg-1/db_dumps
total 0
drwxrwxr-x 2 gpadmin gpadmin 162 Sep 21 12:04 20200921120410
drwxrwxr-x 2 gpadmin gpadmin 119 Sep 21 12:09 20200921120942
drwxrwxr-x 2 gpadmin gpadmin 139 Sep 21 12:10 20200921121014
```
我们继续触发增量备份：
```
[gpadmin@gpmagic ~]$ psql testbackup
psql (9.4.24)
Type "help" for help.

testbackup=# DELETE FROM t3 WHERE random() < 0.001;
DELETE 27
testbackup=# \q
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:17.005078-[INFO]-:Start backup process..................................................................
20200921:12:11:17.005078-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:17.005078-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:11:17.005078-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:11:18.005078-[WARN]-:20200921120942 : Is an invalid backup path and a garbage backup path
20200921:12:11:18.005078-[WARN]-:No backup need merge
20200921:12:11:18.005078-[INFO]-:All table number in database is: [3]
20200921:12:11:18.005078-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921121117/backup_done.log
20200921:12:11:18.005078-[INFO]-:Backup all user table in database: testbackup
20200921:12:11:18.005078-[INFO]-:All table should be backup be specified: [3]
20200921:12:11:18.005078-[INFO]-:Try to execute checkpoint success
20200921:12:11:18.005078-[INFO]-:Last full backup time is: [20200921120410]
20200921:12:11:18.005078-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:11:18.005078-[INFO]-:Ignore table number for increment stat not change is: [2]
20200921:12:11:18.005078-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:11:19.005078-[INFO]-:Number of tables should be backup is: 1
20200921:12:11:19.005078-[INFO]-:Start backup table testbackup.public.t3
20200921:12:11:19.005078-[SUCCESS]-:(1/0/1) testbackup.public.t3 SIZE 71396 BYTES TIME 0 S
20200921:12:11:19.005078-[INFO]-:Finish backup with all success......
[gpadmin@gpmagic ~]$ psql testbackup
psql (9.4.24)
Type "help" for help.

testbackup=# DELETE FROM t3 WHERE random() < 0.001;
DELETE 29
testbackup=# \q
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:37.005450-[INFO]-:Start backup process..................................................................
20200921:12:11:37.005450-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:37.005450-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:11:37.005450-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:11:38.005450-[WARN]-:20200921120942 : Is an invalid backup path and a garbage backup path
20200921:12:11:38.005450-[WARN]-:No backup need merge
20200921:12:11:38.005450-[INFO]-:All table number in database is: [3]
20200921:12:11:38.005450-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921121137/backup_done.log
20200921:12:11:38.005450-[INFO]-:Backup all user table in database: testbackup
20200921:12:11:38.005450-[INFO]-:All table should be backup be specified: [3]
20200921:12:11:38.005450-[INFO]-:Try to execute checkpoint success
20200921:12:11:38.005450-[INFO]-:Last full backup time is: [20200921120410]
20200921:12:11:38.005450-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:11:38.005450-[INFO]-:Ignore table number for increment stat not change is: [2]
20200921:12:11:38.005450-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:11:39.005450-[INFO]-:Number of tables should be backup is: 1
20200921:12:11:39.005450-[INFO]-:Start backup table testbackup.public.t3
20200921:12:11:39.005450-[SUCCESS]-:(1/0/1) testbackup.public.t3 SIZE 71327 BYTES TIME 0 S
20200921:12:11:39.005450-[INFO]-:Finish backup with all success......
[gpadmin@gpmagic ~]$ psql testbackup
psql (9.4.24)
Type "help" for help.

testbackup=# DELETE FROM t3 WHERE random() < 0.001;
DELETE 40
testbackup=# \q
[gpadmin@gpmagic ~]$ gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:54.005822-[INFO]-:Start backup process..................................................................
20200921:12:11:54.005822-[INFO]-:Run command: /usr/local/greenplum-db-6.10.0/bin/gpmcbackup --database testbackup --increment --directory /tmp --residue 2 --compress zstd
20200921:12:11:54.005822-[NOTICE]-:Not specify or out of limit, use default(5): -B
20200921:12:11:54.005822-[NOTICE]-:Increment mode specify, ignore all data filter condition and ignore --truncate
。。。。。。
20200921:12:11:54.005822-[WARN]-:20200921120942 : Is an invalid backup path and a garbage backup path
20200921:12:11:54.005822-[INFO]-:Start increment merge to data: [20200921121014]
20200921:12:11:54.005822-[INFO]-:Merge data alltable size is: [3] and merge size is: [2]
20200921:12:11:54.005822-[INFO]-:All table number in database is: [3]
20200921:12:11:54.005822-[INFO]-:Success log file: /tmp/gpseg-1/db_dumps/20200921121154/backup_done.log
20200921:12:11:54.005822-[INFO]-:Backup all user table in database: testbackup
20200921:12:11:54.005822-[INFO]-:All table should be backup be specified: [3]
20200921:12:11:54.005822-[INFO]-:Try to execute checkpoint success
20200921:12:11:54.005822-[INFO]-:Last full backup time is: [20200921121014]
20200921:12:11:54.005822-[INFO]-:Check heap table stat, if heap table amount is larger, it might cost more time......
20200921:12:11:55.005822-[INFO]-:Ignore table number for increment stat not change is: [2]
20200921:12:11:55.005822-[INFO]-:Backup DDLs and OBJECTs and AUTHORIZATIONs
20200921:12:11:56.005822-[INFO]-:Number of tables should be backup is: 1
20200921:12:11:56.005822-[INFO]-:Start backup table testbackup.public.t3
20200921:12:11:56.005822-[SUCCESS]-:(1/0/1) testbackup.public.t3 SIZE 71229 BYTES TIME 0 S
20200921:12:11:56.005822-[INFO]-:Finish backup with all success......
```
我们再看一下备份目录下的情况：
```
[gpadmin@gpmagic ~]$ ll /tmp/gpseg-1/db_dumps
total 0
drwxrwxr-x 2 gpadmin gpadmin 162 Sep 21 12:11 20200921121014
drwxrwxr-x 2 gpadmin gpadmin 139 Sep 21 12:11 20200921121117
drwxrwxr-x 2 gpadmin gpadmin 139 Sep 21 12:11 20200921121137
drwxrwxr-x 2 gpadmin gpadmin 139 Sep 21 12:11 20200921121154
[gpadmin@gpmagic ~]$ ll /tmp/gpseg0/db_dumps/20200921121014/
total 76
-rw------- 1 gpadmin gpadmin   106 Sep 21 12:10 backup_0_log.log
-rw------- 1 gpadmin gpadmin 10307 Sep 21 12:04 testbackup^0^public.t1.zst
-rw------- 1 gpadmin gpadmin 23183 Sep 21 12:04 testbackup^0^public.t2.zst
-rw------- 1 gpadmin gpadmin 35680 Sep 21 12:10 testbackup^0^public.t3.zst
```
我们看到，最早的一次全量备份的目录已经消失，第一次的无效增量备份的目录也已经消失，而是合并成了一份新的全量目录，这个全量，不是真正执行了一次全量备份，而是根据之前的全量和增量进行的动态合并，是一个虚拟的全量，但是，等同于执行了一次全量备份的操作，从而可以实现，控制增量的个数，避免多次执行全量备份，既保证了备份的尺寸可控，又避免的多次执行全量备份。

