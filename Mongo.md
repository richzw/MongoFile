
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

![](https://cloud.githubusercontent.com/assets/1590890/8564881/e25e3794-2582-11e5-8703-5592ce3360c0.png)

**COMPONENTS**

-  mongod - The database process.
-  mongos - Sharding controller.
-  mongo  - The database shell (uses interactive javascript).

**UTILITIES**

-  mongodump         - MongoDB dump tool - for backups, snapshots, etc..
-  mongorestore      - MongoDB restore a dump
-  mongoexport       - Export a single collection to test (JSON, CSV)
-  mongoimport       - Import from JSON or CSV
-  mongofiles        - Utility for putting and getting files from MongoDB GridFS
-  mongostat         - Show performance statistics

Shards && replica Command
----

    $ mongod --shardsvr --replSet  shard-a  --dbpath  data/rs-a-1 --port 30000 --logpath   data/rs-a-1.log        --fork –nojournal
    $ mongod --configsvr --dbpath data/config-3 --port 27021 --logpath data/config-3.log --fork –nojournal
    $ mongos --configdb ubuntu:27019,ubuntu:27020,ubuntu:27021 --logpath data/mongos.log --fork --port 40000
    $ mongo Ubuntu:30000
    >rs.initiate()
    >rs.add(“Ubuntu:30000”)
    >rs.add(“Ubuntu:30001”, {arbiterOnly: true})
    >rs.conf()
    >rs.status()

Configuration the cluster
-----

$mongo Ubuntu:40000
    
    >sh.addShard(“shard-a/Ubuntu:30000,Ubuntu:30001”)
    >sh.addShard(“shard-b/Ubuntu:30100,Ubuntu:30101”)
    >db.getSiblingDB(“config”).shards.find()

Shard collections

    >sh.enableSharding(“adv_pms_development”) //db name
    >db.getSiblingDB(“config”).databases.find()
    >sh.shardCollection(“adv_pms_development.tasks”, {owner: 1, _id: 1})
    >db.getSiblingDB(“config”).collections.find()

$mongo Ubuntu:40000

    >sh.status()
    >use config 
    
Export and import data

    $mongodump -h ubuntu --port 27017 -d adv_pms_development -c tasks
    $mongorestore -h ubuntu --port 40000 -d adv_pms_development 
                             -c shardtasks dump/adv_pms_development/tasks.bson

GridFS
-----

GridFS is a specification for storing and retrieving files that exceed the BSON-document size limit of __16MB__.

Instead of storing a file in a single document, GridFS divides a file into parts, or chunks, and stores each of those chunks as a separate document. By default GridFS limits chunk size to 256k. GridFS uses two collections to store files. One collection stores the file chunks, and the other stores file metadata.

When you query a GridFS store for a file, the driver or client will reassemble the chunks as needed. You can perform range queries on files stored through GridFS. You also can access information from arbitrary sections of files, which allows you to “skip” into the middle of a video or audio file.

**When to use it?**

In some situations, storing large files may be more efficient in a MongoDB database than on a system-level filesystem.

-	If your filesystem limits the number of files in a directory, you can use GridFS to store as many files as needed.
-	When you want to keep your files and metadata automatically synced and deployed across a number of systems and facilities. 
-	When you want to access information from portions of large files without having to load whole files into memory, you can use GridFS to recall sections of files without reading the entire file into memory.

Do not use GridFS if you need to update the content of the entire file atomically. As an alternative you can store multiple versions of each file and specify the current version of the file in the metadata. You can update the metadata field that indicates “latest” status in an atomic update after uploading the new version of the file, and later remove previous versions if needed.

**Use**

    $ mongofiles --host 10.175.31.248 --port 30000 -d adv_pms_development list
    $ mongofiles --host 10.175.31.248 --port 30000 -d adv_pms_development put/get/delete/search me.jpg
    
**Note**
 
For replica sets, mongofiles can only read from the set’s ‘primary.

Capped Collections
-----

Capped collections are fixed-size collections that support high-throughput operations that insert, retrieve, and delete documents based on insertion order. Capped collections work in a way similar to circular buffers: once a collection fills its allocated space, it makes room for new documents by overwriting the oldest documents in the collection.

Capped collections have the following behaviors:

-	Capped collections guarantee preservation of the insertion order. As a result, queries do not need an index to return documents in insertion order. Without this indexing overhead, they can support higher insertion throughput.
-	Capped collections guarantee that insertion order is identical to the order on disk (natural order) and do so by prohibiting updates that increase document size. Capped collections only allow updates that fit the original document size, which ensures a document does not change its location on disk.
-	Capped collections automatically remove the oldest documents in the collection without requiring scripts or explicit remove operations.

Recommendations and Restrictions

-	You cannot shard a capped collection.
-	Capped collections created after 2.2 have an _id field and an index on the _id field by default.
-	You can update documents in a collection after inserting them; however, these updates cannot cause the documents to grow. If the update operation causes the document to grow beyond their original size, the update operation will fail.

If you plan to update documents in a capped collection, remember to create an index to prevent update operations that require a table scan.

-	You cannot delete documents from a capped collection. To remove all records from a capped collection, use the ‘emptycapped’ command. To remove the collection entirely, use the drop() method.
-	Use natural ordering to retrieve the most recently inserted elements from the collection efficiently. This is (somewhat) analogous to tail on a log file.

**Procedures**

Create Capped Collections

    db.createCollection("mycoll", {capped:true, size:100000})

Query a capped collection

    db.cappedCollection.find().sort( { $natural: -1 } )

Check if a collection is capped

    db.collection.isCapped()

Convert a collection to capped

    db.runCommand({"convertToCapped": "mycoll", size: 100000});



