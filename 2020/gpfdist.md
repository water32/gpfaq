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
LOCATION ('gpfdist://172.28.8.251:8080/dev/shm/lotus.txt')
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
执行外部表的count(*)查询，为了准确的评估，我们进行连续3次查询：
```
lotus=# SELECT count(*) FROM ext_test;
NOTICE:  External scan from gpfdist(s) server will utilize 64 out of 72 segment databases
  count
----------
 89000000
(1 row)

Time: 9655.308 ms
lotus=# SELECT count(*) FROM ext_test;
NOTICE:  External scan from gpfdist(s) server will utilize 64 out of 72 segment databases
  count
----------
 89000000
(1 row)

Time: 9635.492 ms
lotus=# SELECT count(*) FROM ext_test;
NOTICE:  External scan from gpfdist(s) server will utilize 64 out of 72 segment databases
  count
----------
 89000000
(1 row)

Time: 9633.638 ms
```
平均耗时为9641 ms。
```
┌nmon─16g──────[H for help]───Hostname=smdw─────────Refresh= 2secs ───12:49.07────────────────────────────────────────────────────────────────────────────────────────┐
│ CPU Utilisation ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────               │
│---------------------------+-------------------------------------------------+                                                                                       │
│CPU User%  Sys% Wait%  Idle|0          |25         |50          |75       100|                                                                                       │
│  1   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│  2   0.0   0.6   0.0  99.4| >                                               |                                                                                       │
│  3   0.0   1.0   0.0  99.0|>                                                |                                                                                       │
│  4   0.5   3.6   0.0  95.9|s>                                               |                                                                                       │
│  5   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│  6   0.0   0.5   0.0  99.5|>                                                |                                                                                       │
│  7   0.5   0.5   0.5  98.5|>                                                |                                                                                       │
│  8   1.0  12.1   0.0  86.9|ssssss>                                          |                                                                                       │
│  9   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│ 10   1.0   6.1   0.0  92.9|sss>                                             |                                                                                       │
│ 11   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│ 12   1.0   9.5   0.0  89.4|ssss>                                            |                                                                                       │
│ 13   0.5   3.5   0.0  96.0|s>                                               |                                                                                       │
│ 14   0.5   4.5   0.0  94.9|ss  >                                            |                                                                                       │
│ 15   1.0  12.1   0.0  86.9|ssssss>                                          |                                                                                       │
│ 16   1.0  14.1   0.0  84.9|sssssss >                                        |                                                                                       │
│ 17   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│ 18   0.0   3.5   0.0  96.5|s>                                               |                                                                                       │
│ 19   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│ 20   0.0   0.0   0.0 100.0|>                                                |                                                                                       │
│ 21   0.5   5.5   0.5  93.5|ss>                                              |                                                                                       │
│ 22   1.5  12.6   0.0  85.9|ssssss>                                          |                                                                                       │
│ 23   0.5   4.1   0.0  95.3|ss >                                             |                                                                                       │
│ 24   1.0  14.6   0.0  84.4|sssssss>                                         |                                                                                       │
│---------------------------+-------------------------------------------------+                                                                                       │
│Avg   0.4   4.6   0.0  95.0|ss>                                              |                                                                                       │
│---------------------------+-------------------------------------------------+                                                                                       │
│ Network I/O ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────               │
│I/F Name Recv=KB/s Trans=KB/s packin packout insize outsize Peak->Recv Trans                                                                                         │
│      lo      1.2       1.2       4.0    4.0   307.5  307.5       16.1     16.1                                                                                      │
│virbr0-ni     0.0       0.0       0.0    0.0     0.0    0.0        0.0      0.0                                                                                      │
│  virbr0      0.0       0.0       0.0    0.0     0.0    0.0        0.0      0.0                                                                                      │
│     em3      0.0       0.0       0.0    0.0     0.0    0.0        0.0      0.0                                                                                      │
│     em1      0.0       0.0       0.0    0.0     0.0    0.0        0.1      0.0                                                                                      │
│     em2      0.0       0.2       0.5    1.0    64.0  230.0        0.4      0.9                                                                                      │
│     em4      0.0       0.0       0.0    0.0     0.0    0.0        0.0      0.0                                                                                      │
│    p1p1   1792.2  1130760.9  27804.925717.6    66.045023.6     1832.9 1130760.9                                                                                     │
│    p1p2      0.0       0.0       0.0    0.0     0.0    0.0        0.0      0.0                                                                                      │
│──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
从nmon的输出来看，网卡的性能已经达到极限了。接下来，我们使用两个网口进行数据获取操作：
```
DROP EXTERNAL TABLE ext_test;
CREATE READABLE EXTERNAL TABLE ext_test
(
  a text
)
LOCATION (
    'gpfdist://172.28.8.251:8080/dev/shm/lotus.txt',
    'gpfdist://172.28.12.251:8080/dev/shm/lotus.txt'
 )
FORMAT 'TEXT' (DELIMITER 'OFF');
```
接下来，进行连续3次查询：
```
lotus=# SELECT count(*) FROM ext_test;
  count
----------
 89000000
(1 row)

Time: 6529.515 ms
lotus=# SELECT count(*) FROM ext_test;
  count
----------
 89000000
(1 row)

Time: 6631.681 ms
lotus=# SELECT count(*) FROM ext_test;
  count
----------
 89000000
(1 row)

Time: 6335.189 ms
```
平均耗时为6498 ms。
我们通过观察nmon的输出发现，有一个CPU Core已经100%繁忙了，根据我们的经验，我们尝试调整gpfdist的-m参数来增加gpfdist分割数据的效率：
```
[root@smdw ~]# gpfdist -p 8080 -d /../ -s -m 128000
2020-09-19 12:58:06 11018 INFO Before opening listening sockets - following listening sockets are available:
2020-09-19 12:58:06 11018 INFO IPV6 socket: [::]:8080
2020-09-19 12:58:06 11018 INFO IPV4 socket: 0.0.0.0:8080
2020-09-19 12:58:06 11018 INFO Trying to open listening socket:
2020-09-19 12:58:06 11018 INFO IPV6 socket: [::]:8080
2020-09-19 12:58:06 11018 INFO Opening listening socket succeeded
2020-09-19 12:58:06 11018 INFO Trying to open listening socket:
2020-09-19 12:58:06 11018 INFO IPV4 socket: 0.0.0.0:8080
Serving HTTP on port 8080, directory /
```
再次测试，平均耗时为5475 ms。性能已经到了极限。
****
接下来，我们介绍gpfdist的深入优化，首先，我们尝试修改gpfdist源码，加入了异步数据传输和ZSTD压缩传输特性，并使用execute外部表来进行数据获取，我们并没有修改Greenplum的Server端的代码，所以，需要借助execute外部表来完成这个优化测试。测试如下：
```
[root@smdw tmp]# ./gpfdist -p 8080 -d /../ -s -m 440000
2020-09-19 13:03:47 12544 INFO Before opening listening sockets - following listening sockets are available:
2020-09-19 13:03:47 12544 INFO IPV6 socket: [::]:8080
2020-09-19 13:03:47 12544 INFO IPV4 socket: 0.0.0.0:8080
2020-09-19 13:03:47 12544 INFO Trying to open listening socket:
2020-09-19 13:03:47 12544 INFO IPV6 socket: [::]:8080
2020-09-19 13:03:47 12544 INFO Opening listening socket succeeded
2020-09-19 13:03:47 12544 INFO Trying to open listening socket:
2020-09-19 13:03:47 12544 INFO IPV4 socket: 0.0.0.0:8080
Serving HTTP on port 8080, directory /
```
创建新的外部表：
```
CREATE READABLE EXTERNAL WEB TABLE ext_testexcu
(
  a text
)
 EXECUTE E'sh /tmp/gcurl.sh smdw:8080/dev/shm/lotus.txt' 
 FORMAT 'text' (delimiter 'off' null E'\\N' escape E'\\');
```
进行连续3次测试：
```
lotus=# SELECT count(*) FROM ext_testexcu;
  count
----------
 89000000
(1 row)

Time: 4189.400 ms
lotus=# SELECT count(*) FROM ext_testexcu;
  count
----------
 89000000
(1 row)

Time: 4213.136 ms
lotus=# SELECT count(*) FROM ext_testexcu;
  count
----------
 89000000
(1 row)

Time: 4179.077 ms
```
平均耗时为4193 ms。
***
总结

| gpfdist优化情况 | 平均耗时 | 平均加载性能 | 瓶颈点 | gpfdist服务器带宽消耗 |
| :------ | :---: | :---: | :---: | :---: |
  | 无优化 | 9641 ms | 1096 MB/s | 网络带宽 | 1096 MB/s |
| 增加网络带宽 | 6498 ms | 1626 MB/s | CPU主频 | 1626 MB/s |
| -m为128000 | 5475 ms | 1930 MB/s | 网络带宽/CPU主频 | 1930 MB/s |
| 修改源码加入ZSTD压缩传输和异步数据传输 | 4193 ms | 2520 MB/s | CPU主频 | 大约380 MB/s |

对gpfdist进行ZSTD压缩传输和异步数据传输的优化，主要目的是，降低网络带宽的消耗，充分利用多CPU Core的计算能力。尤其是在网络带宽有限的环境，可以发挥极大的性能优势。
