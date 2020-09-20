## 如何评估statement_mem的值

关于一个SQL使用多少内存，在Greenplum中，这是一个很复杂的概念，首先我们根据资源管理的模式来分开讨论，因为，资源队列和资源组的单个Primary的可用内存总量的计算方式是不同的。

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

