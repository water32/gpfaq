gpdbinstall，是一个一键式自动化Greenplum数据库安装初始化脚本，在安装好操作系统和配置好磁盘及网络之后，该脚本可以自动完成接下来的所有工作，直到获得一个可用的Greenplum数据库集群。

下面先来演示一下小的生产规模的集群安装初始化操作，由于测试的集群资源和性能配比欠佳，演示的耗时稍微有点慢。

###此处演示了使用一键式安装脚本完成1+6的集群规模下，初始化72个Primary的过程：

```
[root@smdw gpinstall]# cat pf
  hosts-config    = hosts-config
  gp-admin-name   = gpadmin
  gp-admin-passwd = gpadmin
  gp-version      = 6.9.0
  mas-dev         = /data/gp69
  seg-dev         = /data1/gp69
  seg-dev         = /data2/gp69
  user-id         = 300
  segs-per-disk   = 6
  prefix          = pgpseg
  prefix-master   = gpseg
  prefix-mirror   = mgpseg
  mas-port        = 5432
  port-base       = 40000
  mirror-split    = 0
  mirror-mode     = PAIR
  ignore-hosts
[root@smdw gpinstall]# cat hosts-config
172.28.8.251  :  smdw  :  Master   :  0.01
172.28.8.1    :  sdw1  :  Segment  :  1.01
172.28.8.2    :  sdw2  :  Segment  :  1.02
172.28.8.3    :  sdw3  :  Segment  :  1.03
172.28.8.5    :  sdw5  :  Segment  :  1.04
172.28.8.6    :  sdw6  :  Segment  :  1.05
172.28.8.7    :  sdw7  :  Segment  :  1.06
[root@smdw gpinstall]# ./gpdbinstall --pf pf
20200917:15:13:09.021869-[INFO]-:All command bash,bzip2,curl,expect,ifconfig,ip,less,mkfs.xfs,perl,rsync,sed,ssh,tar,unzip,zip found in current environment
20200917:15:13:09.021869-[INFO]-:Check options legal................
20200917:15:13:09.021869-[WARN]-:Please input password for current user:
123456
20200917:15:13:12.021869-[INFO]-:Start greenplum cluster auto install process...............
20200917:15:13:12.021869-[INFO]-:Run command: ./gpdbinstall --pf pf
20200917:15:13:12.021869-[INFO]-:Check host config................
20200917:15:13:15.021869-[INFO]-:Check password for all machine................
20200917:15:13:19.021869-[INFO]-:Try to install binary file on current machine................
20200917:15:13:19.021869-[INFO]-:Modify system config for greenplum install................
20200917:15:13:22.021869-[INFO]-:Machine 172.28.8.251 smdw find data path: /data/gp69
20200917:15:13:24.021869-[INFO]-:Machine 172.28.8.1 sdw1 find data path: /data1/gp69 /data2/gp69
20200917:15:13:25.021869-[INFO]-:Machine 172.28.8.2 sdw2 find data path: /data1/gp69 /data2/gp69
20200917:15:13:27.021869-[INFO]-:Machine 172.28.8.3 sdw3 find data path: /data1/gp69 /data2/gp69
20200917:15:13:29.021869-[INFO]-:Machine 172.28.8.5 sdw5 find data path: /data1/gp69 /data2/gp69
20200917:15:13:31.021869-[INFO]-:Machine 172.28.8.6 sdw6 find data path: /data1/gp69 /data2/gp69
20200917:15:13:34.021869-[INFO]-:Machine 172.28.8.7 sdw7 find data path: /data1/gp69 /data2/gp69
20200917:15:13:44.021869-[INFO]-:Create segment files................
20200917:15:13:44.021869-[INFO]-:Exchange key for gpadmin
20200917:15:14:19.021869-[INFO]-:Start greenplum initsystem
20200917:15:14:19.021869-[INFO]-:Execute command:
sudo su - gpadmin -c "gpinitsystem -a --lc-collate=C -I /tmp/gpdbinstall/gpinitsystem_config"
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Locale has not been set in , will set to default value
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Locale set to en_US.utf8
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Checking configuration parameters, Completed
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Checking Master host
20200917:15:14:22:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Checking new segment hosts, please wait...

20200917:15:17:16:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Checking new segment hosts, Completed
20200917:15:17:16:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Building the Master instance database, please wait...
20200917:15:17:24:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Starting the Master in admin mode
20200917:15:17:42:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Commencing parallel build of primary segment instances
20200917:15:17:42:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait...
............................................................
20200917:15:17:42:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait...
...............................................................................................................................................................................................................................................................................
20200917:15:22:15:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Spawning parallel processes    batch [2], please wait...
............
20200917:15:22:15:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Waiting for parallel processes batch [2], please wait...
............................................
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:------------------------------------------------
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Parallel process exit status
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:------------------------------------------------
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Total processes marked as completed           = 72
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Total processes marked as killed              = 0
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Total processes marked as failed              = 0
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:------------------------------------------------
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Deleting distributed backout files
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Removing back out file
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-No errors generated from parallel processes
20200917:15:22:59:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Restarting the Greenplum instance in production mode
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Starting gpstop with args: -a -l /home/gpadmin/gpAdminLogs -m -d /data/gp69/default/gpseg-1
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Gathering information and validating the environment...
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Obtaining Segment details from master...
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0'
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='smart'
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Master segment instance directory=/data/gp69/default/gpseg-1
20200917:15:23:00:021863 gpstop:smdw:gpadmin-[INFO]:-Stopping master segment and waiting for user connections to finish ...
server shutting down
20200917:15:23:01:021863 gpstop:smdw:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20200917:15:23:01:021863 gpstop:smdw:gpadmin-[INFO]:-Terminating processes for segment /data/gp69/default/gpseg-1
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Starting gpstart with args: -a -l /home/gpadmin/gpAdminLogs -d /data/gp69/default/gpseg-1
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Gathering information and validating the environment...
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Greenplum Binary Version: 'postgres (Greenplum Database) 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0'
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Greenplum Catalog Version: '301908232'
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Starting Master instance in admin mode
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Obtaining Segment details from master...
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Setting new master era
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Master Started...
20200917:15:23:01:021891 gpstart:smdw:gpadmin-[INFO]:-Shutting down master
20200917:15:23:02:021891 gpstart:smdw:gpadmin-[INFO]:-Commencing parallel segment instance startup, please wait...
.
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-Process results...
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-----------------------------------------------------
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-   Successful segment starts                                            = 72
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-----------------------------------------------------
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-Successfully started 72 of 72 segment instances
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-----------------------------------------------------
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-Starting Master instance smdw directory /data/gp69/default/gpseg-1
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-Command pg_ctl reports Master smdw instance active
20200917:15:23:03:021891 gpstart:smdw:gpadmin-[INFO]:-Connecting to dbname='template1' connect_timeout=15
20200917:15:23:04:021891 gpstart:smdw:gpadmin-[INFO]:-No standby master configured.  skipping...
20200917:15:23:04:021891 gpstart:smdw:gpadmin-[INFO]:-Database successfully started
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Completed restart of Greenplum instance in production mode
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Scanning utility log file for any warning messages
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Log file scan check passed
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Greenplum Database instance successfully created
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-------------------------------------------------------
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-To complete the environment configuration, please
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-update gpadmin .bashrc file with the following
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/data/gp69/default/gpseg-1"
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-   to access the Greenplum scripts for this instance:
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-   or, use -d /data/gp69/default/gpseg-1 option for the Greenplum scripts
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-   Example gpstate -d /data/gp69/default/gpseg-1
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Script log file = /home/gpadmin/gpAdminLogs/gpinitsystem_20200917.log
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-To remove instance, run gpdeletesystem utility
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-To initialize a Standby Master Segment for this Greenplum instance
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Review options for gpinitstandby
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-------------------------------------------------------
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-The Master /data/gp69/default/gpseg-1/pg_hba.conf post gpinitsystem
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-has been configured to allow all hosts within this new
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-array to intercommunicate. Any hosts external to this
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-new array must be explicitly added to this file
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-Refer to the Greenplum Admin support guide which is
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-located in the /usr/local/greenplum-db-6.9.0/docs directory
20200917:15:23:04:023572 gpinitsystem:smdw:gpadmin-[INFO]:-------------------------------------------------------
20200917:15:23:04.021869-[WARN]-:Mirror split value small than 1, no need to add mirrors, skip this step.
20200917:15:23:04.021869-[INFO]-:Add standby master................
20200917:15:23:04.021869-[WARN]-:No standby master should config.
20200917:15:23:08.021869-[WARN]-:Parameter gp_vmem_protect_limit value: Master: 23552 Segment: 8192
Last login: Thu Sep 17 15:23:04 CST 2020
20200917:15:23:21:022234 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c default_statistics_target -v 15 -m 15'
Last login: Thu Sep 17 15:23:08 CST 2020
20200917:15:23:33:022491 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c from_collapse_limit -v 10 -m 10'
Last login: Thu Sep 17 15:23:21 CST 2020
20200917:15:23:45:022672 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_autostats_mode_in_functions -v ON_NO_STATS -m ON_NO_STATS'
Last login: Thu Sep 17 15:23:33 CST 2020
20200917:15:23:57:022844 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_create_table_random_default_distribution -v on -m on'
Last login: Thu Sep 17 15:23:45 CST 2020
20200917:15:24:10:023096 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_enable_relsize_collection -v on -m on'
Last login: Thu Sep 17 15:23:57 CST 2020
20200917:15:24:16:023274 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_max_local_distributed_cache -v 1048576'
Last login: Thu Sep 17 15:24:10 CST 2020
20200917:15:24:28:023515 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_max_partition_level -v 1 -m 1'
Last login: Thu Sep 17 15:24:16 CST 2020
20200917:15:24:36:023721 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_resource_group_memory_limit -v 0.5 -m 0.5'
Last login: Thu Sep 17 15:24:28 CST 2020
20200917:15:24:48:023898 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_segment_connect_timeout -v 180 -m 180'
Last login: Thu Sep 17 15:24:36 CST 2020
20200917:15:24:59:024093 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_compression -v on -m on'
Last login: Thu Sep 17 15:24:48 CST 2020
20200917:15:25:12:024352 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_limit_per_query -v 134217728 -m 134217728'
Last login: Thu Sep 17 15:25:00 CST 2020
20200917:15:25:24:024572 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_limit_per_segment -v 134217728 -m 134217728'
Last login: Thu Sep 17 15:25:12 CST 2020
20200917:15:25:36:024823 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c join_collapse_limit -v 10 -m 10'
Last login: Thu Sep 17 15:25:24 CST 2020
20200917:15:25:43:025000 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c log_min_duration_statement -v 30000 -m 30000'
Last login: Thu Sep 17 15:25:36 CST 2020
20200917:15:25:50:025170 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c log_timezone -v HONGKONG -m HONGKONG'
Last login: Thu Sep 17 15:25:43 CST 2020
20200917:15:26:03:025386 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c max_appendonly_tables -v 50000 -m 50000'
Last login: Thu Sep 17 15:25:50 CST 2020
20200917:15:26:15:025584 gpconfig:smdw:gpadmin-[INFO]:-completed successfully with parameters '-c writable_external_table_bufsize -v 16MB -m 16MB'
[root@smdw gpinstall]#
```
只需要配置好参数，一条命令，就完成了这个集群的初始化工作。检查一下刚初始化的集群情况：
```
[root@smdw gpinstall]# su - gpadmin
Last login: Thu Sep 17 15:26:03 CST 2020
[gpadmin@smdw ~]$ psql -l
                              List of databases
   Name    |  Owner  | Encoding | Collate |   Ctype    |  Access privileges
-----------+---------+----------+---------+------------+---------------------
 postgres  | gpadmin | UTF8     | C       | en_US.utf8 |
 template0 | gpadmin | UTF8     | C       | en_US.utf8 | =c/gpadmin         +
           |         |          |         |            | gpadmin=CTc/gpadmin
 template1 | gpadmin | UTF8     | C       | en_US.utf8 | =c/gpadmin         +
           |         |          |         |            | gpadmin=CTc/gpadmin
(3 rows)

[gpadmin@smdw ~]$ psql postgres
psql (9.4.24)
Type "help" for help.

postgres=# SELECT count(*) FROM gp_segment_configuration;
 count
-------
    73
(1 row)

postgres=# SELECT version();
                                                                                               version

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------
 PostgreSQL 9.4.24 (Greenplum Database 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0) on x86_64-unknown-linux-gnu, compiled by gcc (GCC) 6.4.0, 64-bit compiled
 on Jun 29 2020 22:58:53
(1 row)

postgres=# SELECT pg_postmaster_start_time();
   pg_postmaster_start_time
-------------------------------
 2020-09-17 15:23:03.760003+08
(1 row)

postgres=#
```

###接下来，演示一下两个虚拟机环境安装初始化Greenplum集群的急速过程：
```
[root@sdw001 gpinstall]# cat pf
  hosts-config    = hosts-config
  gp-admin-name   = gpadmin
  gp-admin-passwd = gpadmin
  gp-version      = 6.9.0
  mas-dev         = /data
  seg-dev         = /data
  user-id         = 500
  segs-per-disk   = 1
  prefix          = pgpseg
  prefix-master   = gpseg
  prefix-mirror   = mgpseg
  mas-port        = 5432
  port-base       = 40000
  mirror-split    = 1
  mirror-mode     = PAIR
  password        = 123456
[root@sdw001 gpinstall]# cat hosts-config
  192.168.137.41 : mdw001  :  Master  :   0.01
  192.168.137.42 : sdw002  :  Standby :   0.02
  192.168.137.41 : sdw001  :  Segment :   1.01
  192.168.137.42 : sdw002  :  Segment :   2.02
[root@sdw001 gpinstall]# ./gpdbinstall --pf pf
20200919:10:41:52.001367-[INFO]-:All command bash,bzip2,curl,expect,ifconfig,ip,less,mkfs.xfs,perl,rsync,sed,ssh,tar,unzip,zip found in current environment
20200919:10:41:52.001367-[INFO]-:Check options legal................
20200919:10:41:52.001367-[INFO]-:Start greenplum cluster auto install process...............
20200919:10:41:52.001367-[INFO]-:Run command: ./gpdbinstall --pf pf
20200919:10:41:52.001367-[INFO]-:Check host config................
20200919:10:41:52.001367-[WARN]-:Master ip is same with a segment
20200919:10:41:52.001367-[WARN]-:Standby ip is same with a segment
20200919:10:41:52.001367-[INFO]-:Check password for all machine................
20200919:10:41:52.001367-[INFO]-:Check pairs legal................
20200919:10:41:52.001367-[INFO]-:Try to install binary file on current machine................
20200919:10:41:52.001367-[INFO]-:Modify system config for greenplum install................
20200919:10:41:53.001367-[INFO]-:Machine 192.168.137.41 sdw001 find data path: /data
20200919:10:41:54.001367-[INFO]-:Machine 192.168.137.42 sdw002 find data path: /data
20200919:10:41:55.001367-[INFO]-:Machine 192.168.137.41 sdw001 find data path: /data
20200919:10:41:56.001367-[INFO]-:Machine 192.168.137.42 sdw002 find data path: /data
20200919:10:41:57.001367-[INFO]-:Create segment files................
20200919:10:41:57.001367-[INFO]-:Exchange key for gpadmin
20200919:10:42:02.001367-[INFO]-:Start greenplum initsystem
20200919:10:42:02.001367-[INFO]-:Execute command:
sudo su - gpadmin -c "gpinitsystem -a --lc-collate=C -I /tmp/gpdbinstall/gpinitsystem_config"
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Locale has not been set in , will set to default value
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Locale set to en_US.utf8
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Master IP address array = ::1
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Checking configuration parameters, Completed
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Checking Master host
20200919:10:42:03:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Checking new segment hosts, please wait...
20200919:10:42:05:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Checking new segment hosts, Completed
20200919:10:42:05:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Building the Master instance database, please wait...
20200919:10:42:39:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Starting the Master in admin mode
20200919:10:42:40:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Commencing parallel build of primary segment instances
20200919:10:42:40:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait.....
20200919:10:42:40:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait.........................................
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:------------------------------------------------
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Parallel process exit status
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:------------------------------------------------
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Total processes marked as completed           = 2
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Total processes marked as killed              = 0
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Total processes marked as failed              = 0
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:------------------------------------------------
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Deleting distributed backout files
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Removing back out file
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-No errors generated from parallel processes
20200919:10:43:19:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Restarting the Greenplum instance in production mode
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Starting gpstop with args: -a -l /home/gpadmin/gpAdminLogs -m -d /data/default/gpseg-1
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Gathering information and validating the environment...
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Obtaining Segment details from master...
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0'
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='smart'
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Master segment instance directory=/data/default/gpseg-1
20200919:10:43:19:004616 gpstop:sdw001:gpadmin-[INFO]:-Stopping master segment and waiting for user connections to finish ...
server shutting down
20200919:10:43:20:004616 gpstop:sdw001:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20200919:10:43:20:004616 gpstop:sdw001:gpadmin-[INFO]:-Terminating processes for segment /data/default/gpseg-1
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Starting gpstart with args: -a -l /home/gpadmin/gpAdminLogs -d /data/default/gpseg-1
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Gathering information and validating the environment...
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Greenplum Binary Version: 'postgres (Greenplum Database) 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0'
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Greenplum Catalog Version: '301908232'
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Starting Master instance in admin mode
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20200919:10:43:20:004639 gpstart:sdw001:gpadmin-[INFO]:-Obtaining Segment details from master...
20200919:10:43:21:004639 gpstart:sdw001:gpadmin-[INFO]:-Setting new master era
20200919:10:43:21:004639 gpstart:sdw001:gpadmin-[INFO]:-Master Started...
20200919:10:43:21:004639 gpstart:sdw001:gpadmin-[INFO]:-Shutting down master
20200919:10:43:21:004639 gpstart:sdw001:gpadmin-[INFO]:-Commencing parallel segment instance startup, please wait....
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-Process results...
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-----------------------------------------------------
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-   Successful segment starts                                            = 2
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-----------------------------------------------------
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-Successfully started 2 of 2 segment instances
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-----------------------------------------------------
20200919:10:43:22:004639 gpstart:sdw001:gpadmin-[INFO]:-Starting Master instance sdw001 directory /data/default/gpseg-1
20200919:10:43:23:004639 gpstart:sdw001:gpadmin-[INFO]:-Command pg_ctl reports Master sdw001 instance active
20200919:10:43:23:004639 gpstart:sdw001:gpadmin-[INFO]:-Connecting to dbname='template1' connect_timeout=15
20200919:10:43:23:004639 gpstart:sdw001:gpadmin-[INFO]:-No standby master configured.  skipping...
20200919:10:43:23:004639 gpstart:sdw001:gpadmin-[INFO]:-Database successfully started
20200919:10:43:23:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Completed restart of Greenplum instance in production mode
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Scanning utility log file for any warning messages
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Log file scan check passed
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Greenplum Database instance successfully created
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-------------------------------------------------------
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-To complete the environment configuration, please
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-update gpadmin .bashrc file with the following
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/data/default/gpseg-1"
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-   to access the Greenplum scripts for this instance:
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-   or, use -d /data/default/gpseg-1 option for the Greenplum scripts
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-   Example gpstate -d /data/default/gpseg-1
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Script log file = /home/gpadmin/gpAdminLogs/gpinitsystem_20200919.log
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-To remove instance, run gpdeletesystem utility
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-To initialize a Standby Master Segment for this Greenplum instance
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Review options for gpinitstandby
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-------------------------------------------------------
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-The Master /data/default/gpseg-1/pg_hba.conf post gpinitsystem
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-has been configured to allow all hosts within this new
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-array to intercommunicate. Any hosts external to this
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-new array must be explicitly added to this file
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-Refer to the Greenplum Admin support guide which is
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-located in the /usr/local/greenplum-db-6.9.0/docs directory
20200919:10:43:24:002486 gpinitsystem:sdw001:gpadmin-[INFO]:-------------------------------------------------------
20200919:10:43:24.001367-[INFO]-:Execute command:
sudo su - gpadmin -c "gpaddmirrors -i /tmp/gp_add_mirror_config -a"
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Starting gpaddmirrors with args: -i /tmp/gp_add_mirror_config -a
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0'
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 9.4.24 (Greenplum Database 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0) on x86_64-unknown-linux-gnu, compiled by gcc (GCC) 6.4.0, 64-bit compiled on Jun 29 2020 22:58:53'
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Obtaining Segment details from master...
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Heap checksum setting consistent across cluster
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Greenplum Add Mirrors Parameters
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Greenplum master data directory          = /data/default/gpseg-1
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Greenplum master port                    = 5432
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Parallel batch limit                     = 16
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Mirror 1 of 2
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance host        = sdw001
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance address     = sdw001
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance directory   = /data/default/pgpseg0
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance port        = 40000
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance host         = sdw002
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance address      = sdw002
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance directory    = /data/default/mgpseg0
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance port         = 50000
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Mirror 2 of 2
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance host        = sdw002
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance address     = sdw002
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance directory   = /data/default/pgpseg1
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Primary instance port        = 40000
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance host         = sdw001
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance address      = sdw001
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance directory    = /data/default/mgpseg1
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-   Mirror instance port         = 50000
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:----------------------------------------------------------
20200919:10:43:24:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Starting to modify pg_hba.conf on primary segments to allow replication connections
20200919:10:43:25:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Successfully modified pg_hba.conf on primary segments to allow replication connections
20200919:10:43:25:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-2 segment(s) to add
20200919:10:43:25:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Validating remote directories
20200919:10:43:26:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Configuring new segments
sdw002 (dbid 4): pg_basebackup: base backup completed
sdw001 (dbid 5): pg_basebackup: base backup completed
20200919:10:43:59:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Updating configuration with new mirrors
20200919:10:43:59:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Updating mirrors
20200919:10:43:59:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Starting mirrors
20200919:10:43:59:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-era is 1a2db688d25e3b35_200919104321
20200919:10:43:59:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Commencing parallel segment instance startup, please wait.....
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Process results...
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-******************************************************************
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Mirror segments have been added; data synchronization is in progress.
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Data synchronization will continue in the background.
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-Use  gpstate -s  to check the resynchronization progress.
20200919:10:44:02:004850 gpaddmirrors:sdw001:gpadmin-[INFO]:-******************************************************************
20200919:10:44:08.001367-[INFO]-:Waiting for mirror mode change to Synchronized [4]
20200919:10:44:14.001367-[INFO]-:Waiting for mirror mode change to Synchronized [4]
20200919:10:44:20.001367-[INFO]-:Waiting for mirror mode change to Synchronized [4]
20200919:10:44:26.001367-[INFO]-:Add standby master................
20200919:10:44:57.001367-[WARN]-:Parameter gp_vmem_protect_limit value: Master: 8192 Segment: 8192
Last login: Sat Sep 19 10:44:56 CST 2020 from 192.168.137.41 on pts/3
20200919:10:44:58:005883 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c default_statistics_target -v 15 -m 15'
Last login: Sat Sep 19 10:44:57 CST 2020
20200919:10:44:59:005982 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c from_collapse_limit -v 10 -m 10'
Last login: Sat Sep 19 10:44:58 CST 2020
20200919:10:44:59:006081 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_autostats_mode_in_functions -v ON_NO_STATS -m ON_NO_STATS'
Last login: Sat Sep 19 10:44:59 CST 2020
20200919:10:45:00:006180 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_create_table_random_default_distribution -v on -m on'
Last login: Sat Sep 19 10:45:00 CST 2020
20200919:10:45:01:006278 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_enable_relsize_collection -v on -m on'
Last login: Sat Sep 19 10:45:00 CST 2020
20200919:10:45:01:006377 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_max_local_distributed_cache -v 1048576'
Last login: Sat Sep 19 10:45:01 CST 2020
20200919:10:45:02:006476 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_max_partition_level -v 1 -m 1'
Last login: Sat Sep 19 10:45:01 CST 2020
20200919:10:45:03:006575 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_resource_group_memory_limit -v 0.5 -m 0.5'
Last login: Sat Sep 19 10:45:02 CST 2020
20200919:10:45:03:006674 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_segment_connect_timeout -v 180 -m 180'
Last login: Sat Sep 19 10:45:03 CST 2020
20200919:10:45:04:006773 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_compression -v on -m on'
Last login: Sat Sep 19 10:45:03 CST 2020
20200919:10:45:04:006872 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_limit_per_query -v 134217728 -m 134217728'
Last login: Sat Sep 19 10:45:04 CST 2020
20200919:10:45:05:006971 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c gp_workfile_limit_per_segment -v 134217728 -m 134217728'
Last login: Sat Sep 19 10:45:04 CST 2020
20200919:10:45:06:007069 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c join_collapse_limit -v 10 -m 10'
Last login: Sat Sep 19 10:45:05 CST 2020
20200919:10:45:06:007168 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c log_min_duration_statement -v 30000 -m 30000'
Last login: Sat Sep 19 10:45:06 CST 2020
20200919:10:45:07:007267 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c log_timezone -v HONGKONG -m HONGKONG'
Last login: Sat Sep 19 10:45:06 CST 2020
20200919:10:45:07:007365 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c max_appendonly_tables -v 50000 -m 50000'
Last login: Sat Sep 19 10:45:07 CST 2020
20200919:10:45:08:007464 gpconfig:sdw001:gpadmin-[INFO]:-completed successfully with parameters '-c writable_external_table_bufsize -v 16MB -m 16MB'
```
数据库的部署和初始化迅速完成，并自动根据版本和环境情况进行了参数调整，初始化的是一个有Mirror和Standby的完整环境，且Master和Standby采用与计算节点共机的方式进行部署。
```
[root@sdw001 gpinstall]# su - gpadmin
Last login: Sat Sep 19 10:45:07 CST 2020
[gpadmin@sdw001 ~]$ psql -l
Timing is on.
                              List of databases
   Name    |  Owner  | Encoding | Collate |   Ctype    |  Access privileges
-----------+---------+----------+---------+------------+---------------------
 postgres  | gpadmin | UTF8     | C       | en_US.utf8 |
 template0 | gpadmin | UTF8     | C       | en_US.utf8 | =c/gpadmin         +
           |         |          |         |            | gpadmin=CTc/gpadmin
 template1 | gpadmin | UTF8     | C       | en_US.utf8 | =c/gpadmin         +
           |         |          |         |            | gpadmin=CTc/gpadmin
(3 rows)

[gpadmin@sdw001 ~]$ psql postgres
Timing is on.
psql (9.4.24)
Type "help" for help.

postgres=# SELECT count(*) FROM gp_segment_configuration;
 count
-------
     6
(1 row)

Time: 9.415 ms
postgres=# SELECT version();
                                                                                               version

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------
 PostgreSQL 9.4.24 (Greenplum Database 6.9.0 build commit:ef010af28862a0fed172ca96620cc1037aac71a0) on x86_64-unknown-linux-gnu, compiled by gcc (GCC) 6.4.0, 64-bit co
mpiled on Jun 29 2020 22:58:53
(1 row)

Time: 5.545 ms
```
我们看一下每个实例的情况，可以从下面的输出发现，Primary和Mirror的工作子目录的PREFIX是有区别的，这得益于gpinitsystem的-I参数，可以自由编排所有的目录和镜像关系。
```
postgres=# select * from gp_segment_configuration;
 dbid | content | role | preferred_role | mode | status | port  | hostname | address |        datadir
------+---------+------+----------------+------+--------+-------+----------+---------+-----------------------
    1 |      -1 | p    | p              | n    | u      |  5432 | sdw001   | sdw001  | /data/default/gpseg-1
    2 |       0 | p    | p              | s    | u      | 40000 | sdw001   | sdw001  | /data/default/pgpseg0
    4 |       0 | m    | m              | s    | u      | 50000 | sdw002   | sdw002  | /data/default/mgpseg0
    3 |       1 | p    | p              | s    | u      | 40000 | sdw002   | sdw002  | /data/default/pgpseg1
    5 |       1 | m    | m              | s    | u      | 50000 | sdw001   | sdw001  | /data/default/mgpseg1
    6 |      -1 | m    | m              | s    | u      |  5432 | sdw002   | sdw002  | /data/default/gpseg-1
(6 rows)

Time: 2.454 ms
postgres=#
```
