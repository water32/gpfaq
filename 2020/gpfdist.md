### gpfdist外部表，是Greenplum数据库最重要的数据批量加载技术，有着极高的性能。

本文主要探讨如何来进行性能的调优。
****
首先，我们准备了一个测试数据文件：
```
[gpadmin@smdw ~]$ ll --block-size M /dev/shm/lotus.txt
-rw-r--r--. 1 root root 10567M Jul 27 15:07 /dev/shm/lotus.txt
```
文件尺寸为10567MB，为了确保整个测试过程不受磁盘性能的影响，我们将测试用的数据文件放到内存中，即/dev/shm目录下。
****
首先，我们在一个有72个Primary的集群，创建一个常规的gpfdist外部表，由于GP集群资源有限，设备比较老，这里主要是为了gpfdist的极限性能，所以，只对外部表执行```count(*)```的操作，不会真正的执行数据入库的操作。
```
CREATE READABLE EXTERNAL TABLE ext_test
(
  a text
)
LOCATION ('gpfdist://smdw:8080/dev/shm/lotus.txt')
FORMAT 'TEXT' (DELIMITER 'OFF');
```
启动gpfdist服务，使用简洁模式，指定了路径和端口，其他参数保持缺省值：
```
[root@smdw ~]# . /usr/local/greenplum-db-6.9.0/greenplum_path.sh
[root@smdw ~]# gpfdist -p 8080 -d /../ -s
2020-09-19 12:45:47 7512 INFO Before opening listening sockets - following listening sockets are available:
2020-09-19 12:45:47 7512 INFO IPV6 socket: [::]:8080
2020-09-19 12:45:47 7512 INFO IPV4 socket: 0.0.0.0:8080
2020-09-19 12:45:47 7512 INFO Trying to open listening socket:
2020-09-19 12:45:47 7512 INFO IPV6 socket: [::]:8080
2020-09-19 12:45:47 7512 INFO Opening listening socket succeeded
2020-09-19 12:45:47 7512 INFO Trying to open listening socket:
2020-09-19 12:45:47 7512 INFO IPV4 socket: 0.0.0.0:8080
Serving HTTP on port 8080, directory /
```
执行外部表的count(*)查询
