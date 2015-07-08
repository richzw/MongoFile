
**MongoDB**是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是他支持的查询语言非常强大，
其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

![](https://cloud.githubusercontent.com/assets/1590890/8564721/60877ec0-2581-11e5-9161-4e327215218c.png)

Document -> collections -> dbs

Collection 切分 chunk -> shard. 

**Shard** 意义：scalability and load balance.

MongoDB按shard key，把 collection切割成若干chunks。每个 chunk 的数据结构，是一个三元组，{collection，minKey，maxKey}

**Replica Set** -> availability

**Config servers**用于存储MongoDB集群的元数据 metadata，这些元数据包括如下两个部分，每一个shard server包括哪些chunks，每个chunk存储了哪些 collections 的哪些 documents

