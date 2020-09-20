## 如何评估statement_mem的值

### 作者

陈淼

****

关于一个SQL使用多少内存，在Greenplum中，这是一个很复杂的概念，首先我们根据资源管理的模式来分开讨论，因为，资源队列和资源组的单个Primary的可用内存总量的计算方式是不同的。

**注意：** 我们这里说的一个SQL可使用的内存，指的是每个Primary上的可用内存，并不是整个集群的概念，如果算出的每个Primary上的可用内存是100MB，有100个Primary，那就是每个Primary上都是100MB的可用内存。

****

### 我们先来看一下资源队列的情况

在gp_resource_manager设置为queue时（这也是目前为止，5版本和6版本的缺省设置），单个Primary的可用内存总量，受到gp_vmem_protect_limit参数的限制，该参数的却深知为8192，单位是MB，也就是说，缺省情况下，单个Primary的可用内存总量是8192MB，即8GB，这个缺省值往往与实际情况不符。本文不讨论gp_vmem_protect_limit的合理性，我们讨论单个SQL可以使用多少内存。

在这种模式下，控制单个SQL使用多少内存，通常有2种途径，一种是，通过statement_mem参数来控制，另一种是，通过资源队列的MEMORY_LIMIT属性来控制，如果使用资源队列的MEMORY_LIMIT属性来控制，又会涉及2种分配方式，一种是与ACTIVE_STATEMENTS属性配合，进行平均的分配，另一种是与MAX_COST结合进行加权分配，比如，下面的资源队列的定义：
```
CREATE RESOURCE QUEUE pg_default WITH (ACTIVE_STATEMENTS=20,MAX_COST=-1,MIN_COST=0,COST_OVERCOMMIT=FALSE,PRIORITY=MEDIUM,MEMORY_LIMIT='2000MB');
```
分配到pg_default资源队列的角色，执行SQL时可以获得的内存尺寸为：
```
2000MB ÷ 20 = 100MB
```
如果是与MAX_COST结合，比如下面的资源队列的定义：
```
CREATE RESOURCE QUEUE pg_default WITH (ACTIVE_STATEMENTS=20,MAX_COST=20000000000,MIN_COST=-1,COST_OVERCOMMIT=FALSE,PRIORITY=MEDIUM,MEMORY_LIMIT='-1');
```
而某个SQL的Cost评估是100000000，那么，该SQL可以获得的内存尺寸为：
```
2000MB × 100000000 ÷ 20000000000 = 10MB
```
在这种情况下，一个SQL是否能够获得合适尺寸的内存，完全取决于执行计划评估的是否准确，然而，往往，执行计划对Cost的评估非常的不准确，所以，一般不使用这种内存分配方式。

如果资源队列中，ACTIVE_STATEMENTS属性和MAX_COST属性都是-1，内存的分配完全按照statement_mem参数的值来进行。

****

### 我们先来看一下资源组的情况

在gp_resource_manager设置为group时，单个Primary的可用内存总量，不再受到gp_vmem_protect_limit参数的限制。而是受到下面公式的计算结果的限制：
```
SYS_MEM( = RAM × (vm.overcommit_ratio ÷ 100) + SWAP)
× gp_resource_group_memory_limit
÷ num_of_active_primary
```
其中，SYS_MEM是Primary或者Master当前所在主机的可用内存总量，num_of_active_primary是当前主机的Master和Primary的总个数，gp_resource_group_memory_limit是GUC参数，属于可以通过gpconfig修改的参数，另外两个值是不能随意修改的。

在使用资源组的情况下，又分为2种分配方式，取决于MEMORY_SPILL_RATIO的值是否为0，这个可以是资源组的MEMORY_SPILL_RATIO属性，也可以是GUC参数，当MEMORY_SPILL_RATIO为0时，内存的分配完全按照statement_mem参数的值来进行。

当MEMORY_SPILL_RATIO不为0时，执行SQL时可以获得的内存尺寸为：
```
int(
SYS_MEM( = RAM × vm.overcommit_ratio ÷ 100 + SWAP)
× gp_resource_group_memory_limit
× MEMORY_LIMIT ÷ 100
÷ num_of_active_primary
÷ CONCURRENCY
× MEMORY_SPILL_RATIO ÷ 100
)
```
比如，下面是我的虚拟机环境的情况：
```
[gpadmin@gpmagic ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           3774         176        2132        1147        1466        2168
Swap:          1952           0        1952
[gpadmin@gpmagic ~]$ cat /proc/sys/vm/overcommit_ratio
95
[gpadmin@gpmagic ~]$ psql postgres
psql (9.4.24)
Type "help" for help.
postgres=# show gp_resource_group_memory_limit;
 gp_resource_group_memory_limit
--------------------------------
 0.7
(1 row)
postgres=# select * from gp_segment_configuration;
 dbid | content | role | preferred_role | mode | status | port  | hostname | address |          datadir
------+---------+------+----------------+------+--------+-------+----------+---------+---------------------------
    1 |      -1 | p    | p              | n    | u      |  5432 | gpmagic  | gpmagic | /data/gp6/default/gpseg-1
    2 |       0 | p    | p              | n    | u      | 40000 | gpmagic  | gpmagic | /data/gp6/default/gpseg0
    3 |       1 | p    | p              | n    | u      | 40001 | gpmagic  | gpmagic | /data/gp6/default/gpseg1
(3 rows)
```
资源组的定义为：
```
CREATE RESOURCE GROUP admin_group WITH (CONCURRENCY=10,CPU_RATE_LIMIT=10,MEMORY_SHARED_QUOTA=80,MEMORY_LIMIT=10,MEMORY_SPILL_RATIO=0,MEMORY_AUDITOR='vmtracker');
```
带入公式分别计算MEMORY_SPILL_RATIO为15个16的结果：
```
postgres=# select (3774*95/100+1952) * 0.7 / 3 * 10 /100 / 10 * 15 / 100;
      ?column?
--------------------
 1.9379500000000000
(1 row)
postgres=# select (3774*95/100+1952) * 0.7 / 3 * 10 /100 / 10 * 16 / 100;
      ?column?
--------------------
 2.0671466666666667
(1 row)
```
根据上述的计算结果进行测试：
```
postgres=# set MEMORY_SPILL_RATIO to 15;
SET
postgres=# explain analyze select count(*) from pg_class;
                                                 QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=12.98..12.99 rows=1 width=8) (actual time=0.192..0.192 rows=1 loops=1)
   ->  Seq Scan on pg_class  (cost=0.00..11.78 rows=478 width=0) (actual time=0.007..0.155 rows=478 loops=1)
 Planning time: 3.718 ms
   (slice0)    Executor memory: 56K bytes.
 Memory used:  1024kB
 Optimizer: Postgres query optimizer
 Execution time: 0.250 ms
(7 rows)

postgres=# set MEMORY_SPILL_RATIO to 16;
SET
postgres=# explain analyze select count(*) from pg_class;
                                                 QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=12.98..12.99 rows=1 width=8) (actual time=0.198..0.198 rows=1 loops=1)
   ->  Seq Scan on pg_class  (cost=0.00..11.78 rows=478 width=0) (actual time=0.010..0.158 rows=478 loops=1)
 Planning time: 2.843 ms
   (slice0)    Executor memory: 56K bytes.
 Memory used:  2048kB
 Optimizer: Postgres query optimizer
 Execution time: 0.257 ms
(7 rows)
```
我们看到explain analyze的输出中，Memory used的值是Master的内存使用量，与前面的计算结果一致。

不过，这个内存限制并不是绝对的，比如，计算出的结果为0时：
```
postgres=# select (3774*95/100+1952) * 0.7 / 3 * 10 /100 / 10 * 7 / 100;
        ?column?
------------------------
 0.90437666666666666900
(1 row)

postgres=# set MEMORY_SPILL_RATIO to 7;
SET
postgres=# explain analyze select count(*) from pg_class;
                                                 QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=12.98..12.99 rows=1 width=8) (actual time=0.226..0.226 rows=1 loops=1)
   ->  Seq Scan on pg_class  (cost=0.00..11.78 rows=478 width=0) (actual time=0.011..0.192 rows=478 loops=1)
 Planning time: 2.886 ms
   (slice0)    Executor memory: 56K bytes.
 Memory used:  0kB
 Optimizer: Postgres query optimizer
 Execution time: 0.335 ms
(7 rows)
```
****
最后，还是建议用statement_mem来控制内存的使用情况，而不是使用资源队列的内存限制或者资源组的内存限制，资源队列和资源组主要用于控制并发数。另外，资源组，另一个重要的作用是对CPU资源进行压制。
