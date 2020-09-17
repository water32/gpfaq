```
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


```
