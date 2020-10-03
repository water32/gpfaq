## 全局开窗排序的优化

问题的起因是这样的，国庆长假来了，我们要相约一起打球，可是，我们的大个儿Alex同学被一个Case缠住了，我就想着能不能给他分担一点忧愁，看了一下他的Case，是一个全局排序求row_number()的SQL，SQL本身并不是很复杂，但是，需要对几十亿的数据进行全局排序，因为这个开窗SQL的Over子句中没有Partition By，所以，执行计划只能选择将所有数据Gather到Master来获取全局的序号。

虽然我没有办法很快帮助Alex同学脱困，但是，这个问题还是激发了我的思绪，经过缜密的思考，我发现，是有办法来加速row_number()的获取的，原始的SQL是一个Insert Into ... SELECT row_number() over(Order by ... ) ...这样的形式，实际上，最理想的状态，要避免将全部数据Gather到Master上，于是我就想到了采用分段求值的算法。下面的例子用的是int类型来演示的，对于可以乘除比较大些的类型，都可以用这种方式，而对于日期类型，可以采用trunc的方式来提高时间的最小单位来实现，对于字符串，可以采用截取prefix的方式来实现，对于多字段排序，可以只使用第一个字段来进行分组，本文不再详述。下面演示我的测试：

### 首先，创建一份测试数据

```
drop table if exists win_test;
create table win_test(
    a int,
    b int,
    c int,
    d int,
    e int,
    f int,
    g int,
    h int
)with(appendonly=true,compresstype=zstd,compresslevel=1)distributed randomly;
insert into win_test select round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),
round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1) from generate_series(1,100000);

insert into win_test select round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),
round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1),round(random()*2^31-1) from win_test,generate_series(1,999);

set gp_autostats_mode to none;
set statement_mem to '1999MB';
analyze win_test;
```
### 做一个基准测试

基准测试，只是将测试数据进行一次复制，不做任何处理，以此作为参考，便于对比后续的优化操作，到底有多大的性能提升

```
drop table if exists win_insert;

explain analyze
create table win_insert with(appendonly=true,compresstype=zstd,compresslevel=1) as
select * from win_test
distributed randomly;

Result  (cost=0.00..0.00 rows=0 width=0) (actual time=14.259..9131.050 rows=1392500 loops=1)
  ->  Redistribute Motion 72:72  (slice1; segments: 72)  (cost=0.00..196067.81 rows=1388889 width=32) (actual time=14.255..8160.524 rows=1392500 loops=1)
        ->  Result  (cost=0.00..598.81 rows=1388889 width=36) (actual time=1.177..2050.202 rows=1390997 loops=1)
              ->  Seq Scan on win_test  (cost=0.00..466.14 rows=1388889 width=32) (actual time=1.170..838.714 rows=1390997 loops=1)
Planning time: 11.734 ms
  (slice0)    Executor memory: 442K bytes avg x 72 workers, 442K bytes max (seg0).
  (slice1)    Executor memory: 254K bytes avg x 72 workers, 254K bytes max (seg0).
Memory used:  2046976kB
Optimizer: Pivotal Optimizer (GPORCA)
Execution time: 15752.507 ms
```
下面是基准测试的执行计划

![基准测试的执行计划](https://github.com/water32/gpfaq/blob/master/images/2020/win_tuning/base.png)

### 常规的SQL，不过任何优化

```
drop table if exists win_insert;

explain analyze
create table win_insert with(appendonly=true,compresstype=zstd,compresslevel=1) as
select row_number() over(order by b) row_nmbr,* from win_test
distributed randomly;

Result  (cost=0.00..0.00 rows=0 width=0) (actual time=203015.555..320808.522 rows=1391700 loops=1)
  ->  Result  (cost=0.00..741317.70 rows=1388889 width=40) (actual time=203015.553..320431.284 rows=1391700 loops=1)
        ->  Redistribute Motion 1:72  (slice2; segments: 1)  (cost=0.00..502541.31 rows=100000000 width=40) (actual time=203015.547..319794.239 rows=1391700 loops=1)
              ->  Result  (cost=0.00..493176.87 rows=1388889 width=40) (actual time=203010.981..285649.860 rows=100000000 loops=1)
                    ->  WindowAgg  (cost=0.00..493176.87 rows=1388889 width=40) (actual time=203010.979..271472.583 rows=100000000 loops=1)
                          Order By: b
                          ->  Sort  (cost=0.00..489976.87 rows=1388889 width=32) (actual time=203010.955..230192.336 rows=100000000 loops=1)
                                Sort Key: b
                                Sort Method:  external merge  Disk: 3908768kB
                                ->  Gather Motion 72:1  (slice1; segments: 72)  (cost=0.00..7792.36 rows=100000000 width=32) (actual time=0.095..19334.572 rows=100000000 loops=1)
                                      ->  Seq Scan on win_test  (cost=0.00..466.14 rows=1388889 width=32) (actual time=1.104..374.376 rows=1390997 loops=1)
Planning time: 14.937 ms
  (slice0)    Executor memory: 410K bytes avg x 72 workers, 410K bytes max (seg0).
  (slice1)    Executor memory: 220K bytes avg x 72 workers, 220K bytes max (seg0).
* (slice2)    Executor memory: 1126144K bytes (seg10).  Work_mem: 1126009K bytes max, 10164175K bytes wanted.
Memory used:  2046976kB
Memory wanted:  20328949kB
Optimizer: Pivotal Optimizer (GPORCA)
Execution time: 323975.333 ms
```
下面是常规SQL的执行计划

![常规SQL的执行计划](https://github.com/water32/gpfaq/blob/master/images/2020/win_tuning/pre_tuning.png)

### 优化后的SQL

```
drop table if exists win_insert;

explain analyze
create table win_insert with(appendonly=true,compresstype=zstd,compresslevel=1) as
select s - row_number() over(partition by y.sub order by b desc) + 1 row_nmbr, t.* from win_test t join
(
  select sub, sum(c) over(order by sub) s from (
    select b / 100000 sub,count(*) c from win_test group by sub
  ) x
) y on t.b / 100000 = y.sub
distributed randomly;

Result  (cost=0.00..0.00 rows=0 width=0) (actual time=5674.623..11749.930 rows=1391986 loops=1)
  ->  Result  (cost=0.00..256300.56 rows=1388889 width=40) (actual time=5674.620..11276.185 rows=1391986 loops=1)
        ->  Redistribute Motion 72:72  (slice5; segments: 72)  (cost=0.00..17524.17 rows=1388889 width=48) (actual time=5674.595..9021.595 rows=1391986 loops=1)
              ->  Result  (cost=0.00..17315.50 rows=1388889 width=48) (actual time=6292.626..9669.182 rows=1568496 loops=1)
                    ->  WindowAgg  (cost=0.00..17315.50 rows=1388889 width=48) (actual time=6292.625..9113.850 rows=1568496 loops=1)
                          Partition By: ((win_test.b / 100000))
                          Order By: win_test_1.b
                          ->  Sort  (cost=0.00..17254.39 rows=1388889 width=44) (actual time=6292.576..6611.078 rows=1568496 loops=1)
                                Sort Key: ((win_test.b / 100000)), win_test_1.b
                                Sort Method:  quicksort  Memory: 11640328kB
                                ->  Hash Join  (cost=0.00..10183.88 rows=1388889 width=44) (actual time=4135.394..5101.798 rows=1568496 loops=1)
                                      Hash Cond: (((win_test.b / 100000)) = (win_test_1.b / 100000))
                                      Extra Text: (seg0)   Hash chain length 4655.3 avg, 4834 max, using 325 of 2097152 buckets.
                                      Extra Text: (seg49)  Hash chain length 4654.3 avg, 4850 max, using 337 of 2097152 buckets.
                                      ->  Redistribute Motion 1:72  (slice3)  (cost=0.00..7882.76 rows=100000000 width=12) (actual time=1180.821..1181.218 rows=337 loops=1)
                                            Hash Key: ((win_test.b / 100000))
                                            ->  Result  (cost=0.00..5073.43 rows=1388889 width=12) (actual time=4085.549..4139.090 rows=21475 loops=1)
                                                  ->  WindowAgg  (cost=0.00..5073.43 rows=1388889 width=12) (actual time=4085.548..4133.809 rows=21475 loops=1)
                                                        Order By: ((win_test.b / 100000))
                                                        ->  Gather Motion 72:1  (slice2; segments: 72)  (cost=0.00..3873.43 rows=100000000 width=12) (actual time=4085.468..4102.795 rows=21475 loops=1)
                                                              Merge Key: ((win_test.b / 100000))
                                                              ->  Result  (cost=0.00..1157.10 rows=1388889 width=12) (actual time=3401.758..4042.861 rows=337 loops=1)
                                                                    ->  GroupAggregate  (cost=0.00..1157.10 rows=1388889 width=12) (actual time=3401.757..4042.714 rows=337 loops=1)
                                                                          Group Key: ((win_test.b / 100000))
                                                                          ->  Sort  (cost=0.00..1142.19 rows=1388889 width=4) (actual time=3399.016..3558.457 rows=1568496 loops=1)
                                                                                Sort Key: ((win_test.b / 100000))
                                                                                Sort Method:  quicksort  Memory: 6209032kB
                                                                                ->  Redistribute Motion 72:72  (slice1; segments: 72)  (cost=0.00..499.42 rows=1388889 width=4) (actual time=85.189..1655.176 rows=1568496 loops=1)
                                                                                      Hash Key: ((win_test.b / 100000))
                                                                                      ->  Result  (cost=0.00..482.03 rows=1388889 width=4) (actual time=2.699..1320.175 rows=1390997 loops=1)
                                                                                            ->  Seq Scan on win_test  (cost=0.00..466.14 rows=1388889 width=4) (actual time=2.692..816.209 rows=1390997 loops=1)
                                      ->  Hash  (cost=687.92..687.92 rows=1388889 width=32) (actual time=2825.518..2825.518 rows=1568496 loops=1)
                                            ->  Redistribute Motion 72:72  (slice4; segments: 72)  (cost=0.00..687.92 rows=1388889 width=32) (actual time=0.090..1587.825 rows=1568496 loops=1)
                                                  Hash Key: (win_test_1.b / 100000)
                                                  ->  Seq Scan on win_test win_test_1  (cost=0.00..466.14 rows=1388889 width=32) (actual time=1.243..826.397 rows=1390997 loops=1)
Planning time: 44.614 ms
  (slice0)    Executor memory: 552K bytes avg x 72 workers, 576K bytes max (seg0).
  (slice1)    Executor memory: 220K bytes avg x 72 workers, 220K bytes max (seg0).
  (slice2)    Executor memory: 86419K bytes avg x 72 workers, 90288K bytes max (seg0).  Work_mem: 90105K bytes max.
  (slice3)    Executor memory: 2416K bytes (entry db).
  (slice4)    Executor memory: 224K bytes avg x 72 workers, 224K bytes max (seg0).
  (slice5)    Executor memory: 290844K bytes avg x 72 workers, 311552K bytes max (seg0).  Work_mem: 172025K bytes max.
Memory used:  2046976kB
Optimizer: Pivotal Optimizer (GPORCA)
Execution time: 20205.481 ms
```

下面是优化后SQL的执行计划

![优化后SQL的执行计划](https://github.com/water32/gpfaq/blob/master/images/2020/win_tuning/after_tuning_1.png)

![优化后SQL的执行计划](https://github.com/water32/gpfaq/blob/master/images/2020/win_tuning/after_tuning_2.png)

从优化后的执行计划可以看出，Gather Motion的记录数只有**21475**条，所以，几乎全部的计算工作都是在Segment上完成，并没有像常规SQL那样，把全部(本例为1亿条)记录Gather到Master上。

总体来说：

|  测试场景     | 耗时 | 描述 |
| ---: | ----: | :---- |
| 基准测试 | 15.7 s | 数据从一张表复制到领一张表的基准时间 |
| 常规SQL | 323.9 s | 时间主要都消耗在Master上 |
| 优化SQL | 20.2 s | 极大缩减了Master计算的耗时，只比基准测试多不到30%的时间 |
