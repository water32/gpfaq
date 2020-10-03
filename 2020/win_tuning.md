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

![测试的性能基准](https://github.com/water32/gpfaq/blob/master/images/2020/win_tuning/base.png)






