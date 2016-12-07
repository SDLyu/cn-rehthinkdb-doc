# RethinkDB中的内存使用
这篇文档将想您展示RethinkDB使用内存方式, 以及如何估算并配置缓存大小.

RethinkDB中有3个吃内存的地方:
* 查询以及后台任务, 列如集群中节点之间的backfilling.
* 页面缓存.
* 与数据库大小成正比的metadata.

以下会更详细的说明.

# 缓存大小
RethinkDB会将近期使用的数据保存在内存中以减少对硬盘的访问次数.
默认情况下RethinkDB会自动设置缓存最大容量为`(available_mem - 1024 MB) / 2`.
`available_mem`是当RethinkDB启动时候, 获取物理内存还有多少可用的变量.
如果当可用内存小余1224MB时会自动指定缓存容量为100MB.
您可以在RethinkDB日志或Web管理界面中找出实际的缓存容量限制.

![cache-size](/DocsPages/images/finding-cache-size.png)

大多数情况下自动设置的缓存容量是比较靠谱的, 但您也可以手动条件缓存限制来最大化内存利用率从而达到提高查询性能的目的.

根据RethinkDB启动方式的不同, 调整缓存限制的方式也有所不同:
* bash启动: 您可以在启动参数内加上`--cache-size <limit in MB>`来调整缓存限制, 列如:

  `$ rethinkdb --cache-size 2048`
* 使用配置文件启动: 您可以在配置文件内(`/etc/rethinkdb/instances.d/...`)添加`cache-size=<limit in MB>`来调整缓存限制.

# metadata
RethinkDB可以处理并存储比物理内存大得多的数据库, 但是一些metadata会始终保存在内存中, 以便快速访问.
每个表会占用每个服务器上的8MB内存.

RethinkDB会将数据合并成块, 然后块会以512字节增长, 块最大为4kb. 
块是可以在内存中被释放掉的, 但是每块大约需要10~26字节保存在内存中. 因此内存的开销也会取决于数据块的数量.

其中不超过250字节的小记录会存储在主索引树内, 这样许多小记录就能共享单个4kb块.
大于250字节的较大记录会被存储在单个块中, 大于4kb的记录会被分割成多个4kb块分开存储.

Additional blocks are allocated to store index trees for secondary indexes as well as for the primary index of each table. 
通常每索引每30个记录会用到一个块, 这取决于索引索引键大小.