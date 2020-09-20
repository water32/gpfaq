## 如何评估statement_mem的值

关于一个SQL使用多少内存，在Greenplum中，这是一个很复杂的概念，首先我们根据资源管理的模式来分开讨论，因为，资源队列和资源组的单个Primary的可用内存总量的计算方式是不同的。

**注意：**我们这里说的一个SQL可使用的内存，指的是每个Primary上的可用内存，并不是整个集群的概念，如果算出的每个Primary上的可用内存是100MB，有100个Primary，那就是每个Primary上都是100MB的可用内存。

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
SYS_MEM
× gp_resource_group_memory_limit
÷ num_of_active_primary
```
其中，SYS_MEM是Primary或者Master当前所在主机的可用内存总量，num_of_active_primary是当前主机的Master和Primary的总个数，gp_resource_group_memory_limit是GUC参数，属于可以通过gpconfig修改的参数，另外两个值是不能随意修改的。



