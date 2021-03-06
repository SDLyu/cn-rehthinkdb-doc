# RethinkDB中的限制

RethinkDB中有几项硬限制, 以及几项取决于您机器的软限制.

## 集群/分片限制
* RethinkDB内不会硬限制您的数据库数量.
* RethinkDB内的每个表分片最大上限为64个.

## 表/记录限制
* 单个数据库内的表数量没有硬限制.
* 集群情况下每个表会在每个节点上至少占用10M的磁盘空间.
* 集群情况下当节点上存在副本表时每个副本表会占用8M的内存.
* 对于单个记录大小RethinkDB没有做任何限制. 但是为了性能考虑建议您将记录控制在16M以下.
* 最大的JSON查询被限制在64M
* RethinkDB需要大约总数据大小的1%数据常驻在内存中.

## 键限制

* 主键键值被限制在127个字符内.
* 索引会被大小会被限制在238-主键键大小内. 如果已被索引的记录前面一部分数据相同会导致查询性能下降.
* 当记录值是object或者是`null`时, 记录不会被添加至索引.
* 主键键值可能不会包含`null`codepoint (U+0000).

## 数据类型

* RethinkDB内的数字遵循[IEEE 754](https://zh.wikipedia.org/wiki/IEEE_754)标准. 其能存储−2<sup>53</sup>~2<sup>53</sup>.
如果存储的数字超出了该范围可能会保存与该数所接近的数.
* 默认情况下RethinkDB限制数组内的元素上限为10万个. 您可以调整`run`命令内可选参数`array_limit`来调整上限.

## 其他

使用`orderBy`与`between`排序时会按照字节排序进行排序.
这种情况适用于对UTF-8数据进行排序, 而不适用于对Unicode数据进行排序例如`\u0065\u0301`与`\u00e9`两个看起来都是`é`然而RethinkDB会认为这两个是不同的字符.

某些文件系统下(一般是有压缩或者加密的文件系统)不支持`--direct-io`选项.

默认情况下读操作可能读取到还在内存中还未落盘的数据. 您可以通过设置表的`read_mode`参数来调整这个机制.