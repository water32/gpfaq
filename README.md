## Greenplum知识分享

Greenplum各种知识和技巧

### 作者

陈淼

****

我们将在这里开启一段新的征程，作者后续会以小知识点的形式持续更新Greenplum的相关常见问题和最佳实践经验。

[一键式部署GP集群示例](https://github.com/water32/gpfaq/blob/master/2020/gpdbinstall.md) 介绍了如何使用自己编写的一键式命令完成集群的部署和初始化【**无免费资源提供**】

[并行DDL恢复命令示例](https://github.com/water32/gpfaq/blob/master/2020/gpddlrestore.md) 介绍了如何使用自己编写的高速DDL备份命令和并行DDL恢复命令进行DDL的备份和恢复【**无免费资源提供**】

[gpfdist性能优化介绍](https://github.com/water32/gpfaq/blob/master/2020/gpfdist.md) 介绍2020年中段，我和大牛一起对gpfdist进行性能优化的历程【**无免费资源提供**】

[pgAdmin3可执行文件](https://github.com/water32/pgAdminIII4GP) 【**有可执行文件供下载**】

pgAdmin3主要的修改内容包括：

|  修改内容     | 描述 |
| ---: | :---- |
| 版本兼容性 | 兼容4、5、6版本 |
| 修正资源队列的刷新显示的BUG | 原有的现象，资源队列刷新显示时，就会变为第一个资源队列 |
| 修正资源队列的属性显示问题 | 原有的资源队列属性显示不易读，现已改为可以直接指定的SQL格式 |
| 增加了资源组的显示 | 在5版本和6版本引入了资源组的概念，原有的pgAdmin3无此功能 |
| 修正了函数显示的报错 | 由于5版本和6版本在function方面的系统表修改，导致显示function时报错 |
| CUSTOM外部表的FORMAT格式错误 | 原有的现象，CUSTOM外部表的属性缺少必要的分隔符等，导致DDL无法直接使用 |
| 外部表读写属性未显式的问题 | 原有的现象，显示的DDL信息中无法区分外部表的可读或者可写属性 |
| 物化视图的支持 | 在6.2.1版本开始支持物化视图 |
| 复制分布策略的支持 | 在6版本引入了复制表的策略，需要修改相关逻辑以正确的显示 |
| 显示UNLOGGED属性 | 6版本支持创建UNLOGGED表，而以前的pgAdmin3无法显示该属性 |
| 表空间定义的修正 | 尤其是6版本，表空间不再依赖文件空间，需要修正相关逻辑以正确显示表空间的定义 |
| 外部表图标增强 | 直接从树状图标可以区分可写外部表和可读外部表 |
| 分区表图标增强 | 直接从树状图标可以区分一张表是否有子分区 |
| 权限增强 | 在树状结构中，只显示当前登录角色有权限访问的Schema，Table，View，Function，Sequence等 |
| 国际化问题 | 将一些未正常显示中文的部分进行修正 |
| 图标修正 | 图形化执行计划中，一些图标不能正常显示，比如显示为空白，进行了修复 |

[语句的内存使用量是如何控制的](https://github.com/water32/gpfaq/blob/master/2020/statement_mem.md) 介绍在使用GP时，各种场景下，语句可使用的内存尺寸是如何控制的。

[目前为止最先进的备份命令演示](https://github.com/water32/gpfaq/blob/master/2020/gpmcbackup.md) 【**无免费资源提供**】

[对于全局开窗排序的优化思考](https://github.com/water32/gpfaq/blob/master/2020/win_tuning.md) 通过改写SQL的方式，将全局排序的工作，由Master转移给Segment，极限提升排序性能

