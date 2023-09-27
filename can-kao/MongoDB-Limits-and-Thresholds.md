# MongoDB中的限制与阈值

本文档提供了MongoDB系统的各种硬性和软性限制。

## BSON Documents BSON文档

**BSON文档大小**

  BSON的最大文档大小为16MB。

  最大文档大小有助于确保单个文档不会使用过多的RAM或在传输过程中占用过多的带宽。要存储大于该限制的文档，MongoDB提供了GridFS API。有关GridFS的更多信息，请参阅[mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)和[驱动程序](https://docs.mongodb.com/drivers/)的文档。

**BSON文档的嵌套深度**

  MongoDB支持不超过100层嵌套深度的[BSON文档](https://docs.mongodb.com/manual/reference/glossary/#std-term-document)。

## 命名限制

 **数据库名称的大小写敏感性**

  由于数据库名称在MongoDB中*不区分大小写*，因此数据库名称不能仅因字符的大小写而不同。

**Windows环境下的数据库名称限制**

  对于在Windows上运行的MongoDB环境，数据库名不能包含以下任意一个字符：

  ````bash
  /\. "$*<>:|?
  ````

  另外，数据库名不能包含空字符。
  
 **Unix/Linux系统中的数据库名称限制**

  对于在Unix和Linux系统上运行的MongoDB环境，数据库名不能包含以下任意一个字符：

  ```bash
  `/\. "$`
  ```

  同样的，数据库名不能包含空字符。

**数据库名称的长度**


  数据库名不能为空并且必须小于64个字符。

 **集合名称的限制**

  
  集合名必须以下划线或者字母符号开始，并且*不能*：
  
  - 包含`$`；
  - 为空字符串（比如`""`）；
  - 包含空字符；
  - 以`system.`为前缀（这部分表保留给内部使用）；
  
  如果您的集合名称包含特殊字符（例如下划线字符）或以数字开头，则可以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell中的[db.getCollection()](https://docs.mongodb.com/manual/reference/method/db.getCollection/#mongodb-method-db.getCollection)方法或[驱动程序的类似方法](https://api.mongodb.com/)来访问集合。
  
  命名空间长度：
  
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.4"`**及以上的集群，MongoDB会将对集合/视图名称空间的限制提高到255个字节。对于集合或视图，命名空间包括数据库名称、点号（**`.`**）分隔符和集合/视图名称（例如**`<database>.<collection>`**）;
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为`"4.2"`及以下的集群，集合/视图名称空间的最大长度仍然为120个字节。
  
**字段名称的限制**

  - 字段名称**不能**包含空字符。
  
  - 顶级字段名称**不能**以美元符号（`$`）字符开头。
  
  此外，从MongoDB 3.6开始，服务器允许存储包含点（即`.`）和美元符号（即`$`）的字段名称。
  
    > **重要**
    >
    > MongoDB查询语言无法始终对字段名称包含这些字符的文档查询进行有效地表达（请参阅[SERVER-30575](https://jira.mongodb.org/browse/SERVER-30575)）。
    > 在查询语言添加相关支持之前，建议不要在字段名称中包含`.`和`$`，并且不受MongoDB官方驱动程序支持。
  
  > **警告**
  >
  > **MongoDB does not support duplicate field names**
  >
  > **MongoDB不支持重复的字段名称**
  >
  > MongoDB查询语言对于具有重复字段名称的文档是未定义的。BSON构建器可能支持使用重复的字段名称创建BSON文档。尽管BSON构建器可能不会抛出错误，但是*即使*插入操作返回成功，也不支持将这些文档插入MongoDB。例如，通过MongoDB驱动程序插入具有重复字段名称的BSON文档可能会导致驱动程序在插入之前静默删除重复值。

## 命名空间

**命名空间长度**

  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.4"`**及以上的环境，MongoDB会将对集合/视图名称空间的限制提高到255个字节。对于集合或视图，命名空间包括数据库名称、点号（**`.`**）分隔符和集合/视图名称（例如**`<database>.<collection>`**）;
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.2"`**及以下的环境，集合/视图名称空间的最大长度仍然为120个字节。

  > **提示**
  >
  > 另请参考：[命名限制](https://docs.mongodb.com/manual/reference/limits/#std-label-faq-restrictions-on-collection-names)


## 索引

**索引键的限制**

  > **注意**
  >
  > **4.2版本有变更**
  >
  > 从4.2版本开始，MongoDB对于将[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置成**`"4.2"`**及以上的环境去除了此[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)。
  
  对于从MongoDB 2.6到将fCV设置为**`"4.2"`**或更早的MongoDB版本，索引条目的*总大小*必须*小于*1024字节，该总大小可能包括结构体开销，具体取决于BSON类型。
  
  当[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)存在时：
  
  - 如果现有文档的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则MongoDB**不会**在集合上创建索引。
  - 如果索引字段的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则重新索引操作将出错。重新索引操作是[compact]()命令以及[db.collection.reIndex()]()方法的一部分，因为这些操作会删除集合中的*所有*索引，然后按顺序重新创建它们，所以[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)中的错误阻止了这些操作的重建集合的所有剩余索引。
  - MongoDB不会将任何具有索引字段的文档插入到索引集合中，该文档的索引字段的对应索引条目将超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，而是将返回错误。MongoDB的早期版本将插入此类文档，但不会为其创建索引。
  - 如果更新的值导致索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则对索引字段的更新将出错。如果现有文档包含索引条目超过该限制的索引字段，则导致该文档在磁盘上重新定位的*任何*更新都将返回错误。
  - [mongorestore](https://docs.mongodb.com/database-tools/mongorestore/#mongodb-binary-bin.mongorestore)和[mongoimport](https://docs.mongodb.com/database-tools/mongoimport/#mongodb-binary-bin.mongoimport)将不会插入包含索引字段的文档，该字段的相应索引条目将超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)。
  - 在MongoDB 2.6中，如果该索引字段的对应索引条目在初始同步时超出了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，副本集的从节点将继续复制带有索引字段的文档，但会在日志中显示警告信息。从节点还允许对包含了对应的索引条目超过了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)的索引字段的集合进行索引构建和重建操作，但在日志中显示警告信息。使用混合版本副本集（其中次要版本为2.6和主版本为版本2.4），从节点将复制在2.4主版本上插入或更新的文档，但是如果文档包含一个索引字段（其对应的索引条目超过了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)），则会在日志中显示错误消息。
  - 对于现有分片集合，如果块中包含文档的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)的索引字段，则块迁移将失败。
  
**每个集合中的索引个数**

  单个集合内*不能超过*64个索引。

**索引名称长度**

  > **注意**
  >
  > **4.2版本有变更**
  >
  > 从4.2版本开始，MongoDB对于将[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置成**`"4.2"`**及以上的环境去除了此[索引名称长度限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)。

  在将fCV设置为**`"4.0"`**及以下的MongoDB或MongoDB的早期版本中，标准的索引名称，包括名称空间和点分隔符（即`<database name>.<collection name>.$<index name>`），不能超过127个字节。

  默认情况下，`<index name>`是字段名称和索引类型的串联。您可以为`createIndex()`方法显式指定`<index name>`，以确保标准索引名称不超过限制。

**复合索引的字段数量**

  复合索引中所包含的字段不能超过32个。

**查询不能同时使用文本索引和地理空间索引**

  您不能将需要特殊[文本索引](https://docs.mongodb.com/manual/core/index-text/#std-label-create-text-index)的[$text](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)查询与需要不同类型特殊索引的查询运算符组合在一起。例如，您不能将[$text](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)查询与[$near](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)运算符结合使用。

**具有2dsphere索引的字段只能保存几何数据**

  具有[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)索引的字段必须以[坐标对](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs)或[GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON)数据的形式保存几何数据。如果您尝试在**2dsphere**索引字段中插入包含非几何数据的文档，或者在索引字段包含非几何数据的集合上构建**2dsphere**索引，则该操作将失败。

  > **提示**
  >
  > **另请参考：**
  >
  > [分片操作限制](https://docs.mongodb.com/manual/reference/limits/#std-label-limits-sharding-operations)中的唯一索引限制

**WiredTiger存储引擎从覆盖查询返回的NaN值始终为double类型**

  如果从索引覆盖的查询返回的字段的值为**NaN**，则该**NaN**值的类型*始终*为**double**。

**多键索引**

  多键索引不能覆盖对数组字段的查询。

**地理位置索引**

  地理位置索引无法覆盖查询。

**索引构建中的内存使用情况** 

  [createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)支持在集合上构建一个或多个索引。[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)使用内存和磁盘上的临时文件的组合来完成索引构建。[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)的内存使用量的默认限制是200MB（对于4.2.3和更高版本）和500MB（对于4.2.2和更早版本），这是使用单个[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)命令构建的所有索引之间共享的。一旦达到内存限制，[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)将使用[--dbpath](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--dbpath)指定的目录中名为`_tmp`子目录中的临时磁盘文件来完成构建。

  您可以通过设置[maxIndexBuildMemoryUsageMegabytes](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes)这一服务器参数来覆盖该内存限制。设置更高的内存限制可能会导致索引构建更快地完成。但是，相对于系统上未使用的RAM设置此限制过高会导致内存耗尽和MongoDB服务停止。

  *4.2版本有更新*

  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.2"`**的环境，索引创建的内存限制对所有索引创建生效；
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.0"`**的环境，索引创建的内存限制仅对前台建索引生效；

  可以通过诸如[创建索引](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/)之类的用户命令或诸如[初始化同步](https://docs.mongodb.com/manual/core/replica-set-sync/)之类的管理过程来启动索引构建。两者均受[maxIndexBuildMemoryUsageMegabytes](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes)设置的限制。

  [初始化同步操作](https://docs.mongodb.com/manual/core/replica-set-sync/)一次仅填充一个集合，并且没有超过内存限制的风险。但是，用户可能会同时在多个数据库中的多个集合上启动索引构建，并且可能消耗的内存量大于[maxIndexBuildMemoryUsageMegabytes]()中设置的限制。

  >**提示**
  >
  >为了最大程度地减少在副本集和具有副本集分片的分片集群上建立索引的影响，请使用滚动索引生成过程，如[在副本集上滚动索引构建](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/)所述。

**字节序和索引类型**

  以下索引类型仅支持简单的二进制比较规则而不支持[字节序](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation)：

  - [文本](https://docs.mongodb.com/manual/core/index-text/)索引;
  - [2d](https://docs.mongodb.com/manual/core/2d/)索引；
  - [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/)索引。

  > **提示**
  >
  > 为了在一个包含非简单字节序的集合上创建一个`text`，`2d`或`geoHaystack`索引，您必须在创建索引时显示指定`collation: {locale: "simple"}`。

**隐藏索引**

  - 你无法[隐藏](https://docs.mongodb.com/manual/core/index-hidden/)`_id`索引。
  - 在[隐藏索引](https://docs.mongodb.com/manual/core/index-hidden/)上无法使用[hint()](https://docs.mongodb.com/manual/reference/method/cursor.hint/#mongodb-method-cursor.hint)

## 文档数

**限制集合中的最大文档数量**
  
  如果使用`max`参数为限制集合指定最大文档数，则该限制必须少于`2^32`个文档。如果在创建上限集合时未指定最大文档数，则对文档数没有限制。

## 副本集

**副本集成员个数**

副本集能拥有不超过50个成员。

**副本集中可投票成员个数**

  副本集最多可以有7个投票成员。有关成员总数超过7个的副本集，请参阅[非投票成员](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-non-voting-members)。

**自动创建的oplog表的最大大小**
  
  如果您未明确指定oplog表的大小（即使用[oplogSizeMB](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.oplogSizeMB)或[--oplogSize](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--oplogSize)），则MongoDB将创建一个不超过50GB的oplog表。[1]
  
  > [1]从MongoDB 4.0开始，操作日志可以超过其配置的大小限制，以避免删除[大多数提交点](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#mongodb-data-replSetGetStatus.optimes.lastCommittedOpTime)。

## 分片集群

分片群集具有此处描述的限制和阈值。

### 分片操作限制

**分片环境中无法执行的操作**

  The [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch) command is not supported in sharded environments.
  [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) 不允许从[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) 函数引用db对象。这在未分片的集合中并不常见。

  分片环境不支持[geoSearch](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)命令。

**分片集群中的覆盖索引**

  从MongoDB 3.0开始，如果索引不包含分片键，则对于运行在[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)上的查询而言，索引不能[覆盖](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries)[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)集合上的查询，但`_id`索引除外：如果分片集合上的查询仅指定条件在`_id`字段上并仅返回`_id`字段，即使`_id`字段不是分片键，`_id`索引也可以覆盖查询。

  在以前的版本中，对于运行在[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)上的查询而言，索引无法[覆盖](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries)[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)集合上的查询。

对已存在的集合进行分片的数据大小限制

  如果现有集合的大小未超过特定限制，则只能对其进行分片。可以基于所有分片键值的平均大小以及配置的块大小来估计这些限制。
  
  >  **重要**
  >
  >  这些限制仅适用于初始化分片操作。成功启用分片后，分片集合可以增长到任何大小。

  如果如下的公式来计算*理论*最大集合大小。

  ```bash
  maxSplits = 16777216 (bytes) / <average size of shard key values in bytes> maxCollectionSize (MB) = maxSplits * (chunkSize / 2)
  ```

  > **注意**
  >
  > BSON文档的最大大小为`16MB`或者`16777216B`。
  >
  > 所有的转换都是基于二进制的，比如1024KB = 1MB。


如果**maxCollectionSize**小于或几乎等于目标集合，则增加块大小以确保成功进行初始分片。如果对计算结果是否过于“接近”目标集合大小有疑问，最好增加块大小。

成功完成初始化分片后，您可以根据需要减小块大小。如果以后减小块大小，则所有块可能都需要花费一些时间才能拆分为新的大小。有关修改块大小的说明，请参阅[修改分片群集中的块大小](https://docs.mongodb.com/manual/tutorial/modify-chunk-size-in-sharded-cluster/)。

下表使用上述公式说明了最大的集合规模： 

| 分片键值的平均大小              | 512字节 | 256字节 | 128字节 | 64字节 |
| :------------------------------ | ------- | ------- | ------- | ------ |
| **拆分的最大次数**              | 32768   | 65536   | 131072  | 262144 |
| **最大集合大小（块大小64MB）**  | 1TB     | 2TB     | 4TB     | 8TB    |
| **最大集合大小（块大小128MB）** | 2TB     | 4TB     | 8TB     | 16TB   |
| **最大集合大小（块大小256MB）** | 4TB     | 8TB     | 16TB    | 32TB   |

- `Single Document Modification Operations in Sharded Collections` **分片集合中的单文档修改操作**
  
  指定了`justOne`或`multi：false`选项的分片集合的所有[`update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update)和[`remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#mongodb-method-db.collection.remove)操作必须在查询条件中包括[分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)或`_id`字段。否则将返回错误。

**分片集合中的唯一索引**
  
  MongoDB不支持跨分片的唯一索引，除非唯一索引包含完整的分片键作为索引前缀。在这些情况下，MongoDB将在整个索引键上而不是单个字段上进行唯一性约束。
  
  > **提示**
  >
  > 替代方法请参考[任意字段的唯一性约束](https://docs.mongodb.com/manual/tutorial/unique-constraints-on-arbitrary-fields/#std-label-shard-key-arbitrary-uniqueness)。

**迁移时每个块的最大文档数量**
  
  默认情况下，如果块中的文档数大于配置的[块大小](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-sharding-chunk-size)除以平均文档大小所得结果的1.3倍，则MongoDB无法移动该块。[`db.collection.stats()`](https://docs.mongodb.com/manual/reference/method/db.collection.stats/#mongodb-method-db.collection.stats)的返回结果包含了`avgObjSize`字段，该字段表示集合中的平均文档大小。
  
  对于[太大而无法迁移](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#std-label-migration-chunk-size-limit)的块，从MongoDB 4.4开始：
  
  - 新的平衡器设置——`tryToBalanceJumboChunks`允许平衡器迁移过大而无法移动的块，只要这些块未标记为[巨型(Jubmo)](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk)即可。有关详细信息请参见[均衡超出大小限制的块](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-balance-chunks-that-exceed-size-limit)。
  - [`moveChunk`](https://docs.mongodb.com/manual/reference/command/moveChunk/#mongodb-dbcommand-dbcmd.moveChunk)命令可以指定一个新选项[forceJumbo](https://docs.mongodb.com/manual/reference/command/moveChunk/#std-label-movechunk-forceJumbo)，以允许迁移过大而无法移动的块，无论该块有没有被标记为[巨型(Jubmo)](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk)。

### 分片键限制

**分片键大小**

  从4.4版本开始，MongoDB去除了关于分片键大小的限制。

  在4.2及之前的版本，一个分片键大小不能超过512B。

**分片键索引类型**

  [分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)索引可以是分片键上的升序索引，也可以是以分片键开头并为分片键指定升序的复合索引，也可以是[哈希索引](https://docs.mongodb.com/manual/core/index-hashed/)。

  [分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)索引不能是在[分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)字段上指定的[多键索引](https://docs.mongodb.com/manual/core/index-multikey/)，[文本索引](https://docs.mongodb.com/manual/core/index-text/)或[地理空间索引](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial)。

**分片键在MongoDB4.2及以前的版本中是不可改变的**

  > **注意**
  >
  > **4.4版本中更新**
  >
  > 从MongoDB 4.4开始，您可以通过向现有键添加一个或多个后缀字段来优化集合的分片键。请参阅[fineCollectionShardKey](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#mongodb-dbcommand-dbcmd.refineCollectionShardKey)。

  在MongoDB 4.2和更早版本中，一旦对集合进行分片，则分片键是不可改变的。也就是说，您不能为该集合选择其他分片键。

  如果必须更改分片键（则需要进行以下的重建步骤）：

  - 将MongoDB中的所有数据转储为外部格式。
  - 删除原始分片集合。
  - 使用新的分片密钥配置分片。
  - 对分片建范围进行[预分片](https://docs.mongodb.com/manual/tutorial/create-chunks-in-sharded-cluster/)以确保初始均匀分配。
  - 将转储的数据还原到MongoDB中。

**单调递增的分片键会限制插入性能**

  对于具有高插入量的集群，具有单调递增和递减性质的分片键可能会影响插入的吞吐量。如果您的分片键是`_id`字段，请注意`_id`字段的默认值是通常具有递增值的[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)。
  
  当使用单调递增的分片键进行插入文档操作时，所有的插入都落在单个[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)上的同一[块](https://docs.mongodb.com/manual/reference/glossary/#std-term-chunk)。系统最终划分接收所有写操作的块范围，并迁移其内容以更均匀地分配数据。但是，群集在任何时候都只将插入操作定向到单个分片，这会造成插入吞吐量的瓶颈。
  
  如果集群上的操作主要是读取操作和更新，则此限制可能不会影响集群。
  
  为避免此约束，请使用[哈希分片键](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding)或选择一个不会单调增加或减少的字段。
  
  [哈希分片键](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding)和[哈希索引](https://docs.mongodb.com/manual/core/index-hashed/#std-label-index-type-hashed)存储具有升序值的键的哈希值。

## 操作

**排序操作**
  
  如果MongoDB无法使用一个或多个索引来获取排序顺序，则MongoDB必须对数据执行阻塞式排序操作。该名称指的是`SORT`阶段在返回任何输出文档之前读取所有输入文档的要求，从而阻止了该特定查询的数据流。
  
  如果MongoDB要求使用100MB以上的系统内存进行阻塞排序操作，则除非查询指定[`cursor.allowDiskUse()`](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse)（*MongoDB 4.4中的新增功能*），否则MongoDB将返回错误。[allowDiskUse](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse)允许MongoDB在处理阻塞排序操作时使用磁盘上的临时文件来存储超过100MB系统内存限制的数据。
  
   *在版本4.4中进行了更改*：对于MongoDB 4.2和更低版本，阻塞排序操作不能超过32MB系统内存。
  
  有关排序和索引使用的更多信息，请参见[排序和索引使用](https://docs.mongodb.com/manual/reference/method/cursor.sort/#std-label-sort-index-use)。
  
**聚合管道操作**

  流水线级的RAM限制为100MB。如果阶段超出此限制，则MongoDB将产生错误。要允许处理大型数据集，请使用`allowDiskUse`选项启用聚合管道阶段以将数据写入临时文件。

  *在版本3.4中进行了更改*。

  [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)阶段必须保持在100 MB内存限制内。如果为[`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)操作指定了`allowDiskUse:true`，则[`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)阶段将忽略该选项。如果[`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)操作中还有其他阶段，则`allowDiskUse:true`选项对这些其他阶段有效。

  从MongoDB 4.2开始，[事件探查器日志消息]()和[诊断日志消息]()均包含`usedDisk`字段，其指示了是有否有聚合阶段由于[内存限制](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#std-label-agg-memory-restrictions)而将数据写入磁盘上临时文件。

  > **提示**
  >
  > **另请参考：**
  >
  > - [`$sort`与内存限制](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#std-label-sort-memory-limit)
  > - [`$group`操作符与内存](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#std-label-group-memory-limit)

**聚合以及读关注**

  - 从MongoDB 4.2开始，[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out)阶段不能与[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)指定[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注，则不能在管道中包括[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out)阶段。
  -  [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段不能与[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)指定[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)读取关注点，则不能在管道中包括[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段。

**2d地理位置查询无法使用`$or`操作符**

  > **提示**
  >
  > **查看: 参考：**
  >
  > - [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#mongodb-query-op.-or)
  > - [`2d` Index Internals](https://docs.mongodb.com/manual/core/geospatial-indexes/)

**地理位置查询**

  对于地理位置查询，使用`dusphere`索引的结果。

  将`2d`索引用于球形查询可能会导致错误的结果，例如将`2d`索引用于环绕两极的球形查询。

**地理空间坐标**

  - 有效的经度值在-180到180之间（包括两者）。
  - 有效的纬度值在-90到90之间（包括两者）。

**GeoJSON多边形的面积**

  对于[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects)或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)，如果您指定面积大于单个半球的单环多边形，则[在$ geometry表达式中包括自定义MongoDB坐标参考系统](https://docs.mongodb.com/manual/reference/operator/query/geometry/#mongodb-query-op.-geometry)； 否则，[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) 或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) 将查询互补几何。对于面积大于半球的所有其他GeoJSON多边形，[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) 或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) 查询互补几何。

**多文档事务**

  对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)而言：

  - 您可以在**现有**集合上指定读/写（CRUD）操作。有关CRUD操作的列表，请参阅[CRUD操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud)。

  - 使用[fcv](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)**`“4.4”`**或更高版本时，可以在事务中创建集合和索引。有关详细信息，请参见[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

  - 事务中使用的集合可以位于不同的数据库中。

    > **注意** 
    >
    > 您无法在跨分片写入事务中创建新集合。例如，如果您在一个分片中写入现有集合，而在另一个分片中隐式创建一个集合，则MongoDB无法在同一事务中执行这两项操作。

  - 您无法写[限制（capped）](https://docs.mongodb.com/manual/core/capped-collections/)集合。（从MongoDB 4.2开始）

  - 您无法在`config`，`admin`或`local`数据库中读取/写入集合。

  -  您无法写入`system.*`集合。

  - 您无法返回受支持操作的查询计划（即`explain`）。

  - 对于在事务外部创建的游标，不能在事务内部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。对于在事务中创建的游标，不能在事务外部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。从MongoDB 4.2开始，您不能将 [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)指定为[事务](https://docs.mongodb.com/manual/core/transactions/)中的第一个操作。

  *4.4版本中有更新*

  以下操作在事务中不被允许：

  - 影响数据库目录的操作，例如在使用[fcv](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)**`"4.2"`**或更低版本时创建/删除集合或索引。使用fcv**`"4.4"`**或更高版本时，您可以在事务中创建集合和索引，除非该事务是跨分片写入事务。有关详细信息，请参考[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。
  - 在跨分片写入事务中创建新集合。例如，如果您在一个分片中写入现有集合，而在另一个分片中隐式创建一个集合，则MongoDB无法在同一事务中执行这两项操作。
  - 当使用除[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)以外的其他读关注级别时[显示创建集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)，如 [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)方法；以及显示创建索引，如[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) 和 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) 方法。
  - [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections) 和 [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes)命令及其辅助方法。
  - 其他非CRUD和非信息性操作，例如[`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count)等及其辅助方法。

  事务存在一个生命周期限制，由[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)指定，默认值为60s。

**批量写大小限制**

  在单个批处理操作中允许`100,000`次[写入](https://docs.mongodb.com/manual/reference/command/nav-crud/)，这由对服务器的单个请求定义。

  *在3.6版中进行了更改*：写入限制从`1,000`增加到`100,000`。此限制也适用于旧式`OP_INSERT`消息。

  [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell中的[`Bulk()`](https://docs.mongodb.com/manual/reference/method/Bulk/#mongodb-method-Bulk) 操作和驱动程序中的类似方法没有此限制。

**视图**

  视图定义`管道`不能包含 [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) 或者 [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) 阶段。如果视图定义包括嵌套管道（例如，视图定义包括[`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup) 或者[`$facet`](https://docs.mongodb.com/manual/reference/operator/aggregation/facet/#mongodb-pipeline-pipe.-facet) 阶段），则此限制也适用于嵌套管道。

  视图具有以下操作限制：

  - 只读
  - 不能重命名
  - 不支持以下[投射](https://docs.mongodb.com/manual/reference/operator/projection/)操作符：[$](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)、[$elemMatch](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch)、[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)、[$meta](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta)
  - 不支持文本索引
  - 不支持map-reduce操作
  - 不支持geoNear操作（即[$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)管道阶段）

**投射限制**

*4.4版的新功能:*

**$前缀的字段路径限制**

  从MongoDB 4.4开始， [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) 无法投射以`$`开头的字段，但[DBRef字段](https://docs.mongodb.com/manual/reference/database-references/#std-label-dbref-explanation)除外。例如，从MongoDB 4.4开始，以下操作无效：

  ```js
  db.inventory.find( {}, { "$instock.warehouse": 0, "$item": 0, "detail.$price": 1 } ) // Invalid starting in 4.4
  ```

MongoDB已经有一个[限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names)，即顶级字段名称不能以美元符号（`$`）开头。在早期版本中，MongoDB忽略`$`前缀的字段投射。

**$位置运算符的放置限制**


  从MongoDB 4.4开始，[`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)投射运算符只能出现在字段路径的末尾。例如`"field.$"`或`"fieldA.fieldB.$"`。例如，从MongoDB 4.4开始，以下操作无效： 

  ```js
  db.inventory.find( { }, { "instock.$.qty": 1 } ) // Invalid starting in 4.4
  ```

  要解决此问题，请删除`$`投射运算符后面的字段路径部分。在以前的版本中，MongoDB会忽略`$`后面的路径部分； 即，该投射被视为`"instock.$"`。

**空字段名称投射限制**

从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify)不能包含空字段名称的投射。例如，从MongoDB 4.4开始，以下操作无效： 

  ```js
  db.inventory.find( { }, { "": 0 } ) // Invalid starting in 4.4
  ```

  在以前的版本中，MongoDB将空字段的包含/排除视为不存在字段的投射。

**路径冲突：嵌入式文档及其字段**

  从MongoDB 4.4开始，使用嵌入文档的任何字段来投射嵌入文档都是非法的，例如，考虑包含文档的集合`inventory`，其中包含`size`字段： 

  ```js
  { ..., size: { h: 10, w: 15.25, uom: "cm" }, ... }
  ```

  从MongoDB 4.4开始，以下操作因`路径冲突`错误而失败，因为它尝试同时投射`size`文档和`size.uom`字段： 

  ```js
  db.inventory.find( {}, { size: 1, "size.uom": 1 } )  // Invalid starting in 4.4
  ```

  在以前的版本中，嵌入文档及其字段之间的最后一个投射决定了整个投射：

  - 如果嵌入式文档的投射紧随其字段的所有投射之后，则MongoDB会投射嵌入式文档。例如，投射文档`{"size.uom"：1, size：1}`产生与投射文档`{size：1}`相同的结果。
  - 如果嵌入式文档的投射先于其任何字段的投射，则MongoDB会投射指定的一个或多个字段。例如，投射文档`{"size.uom"：1, size：1,"size.h"：1}`产生与投射文档`{"size.uom"：1, "size.h":1}`相同的结果。

**路径冲突：数组和嵌入式字段的`$slice`**

  从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify)投射不能同时包含数组的[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)和数组中嵌入的字段，例如，考虑包含数组字段`instock`的集合`inventory`： 

  ```js
  { ..., instock: [ { warehouse: "A", qty: 35 }, { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ], ... }
  ```

  从MongoDB 4.4开始，以下操作会因`路径冲突`而失败：

  ```js
  db.inventory.find( {}, { "instock": { $slice: 1 }, "instock.warehouse": 0 } ) // Invalid starting in 4.4
  ```

  在以前的版本中，投射会同时应用这两个投射并返回`instock`数组中的第一个元素（`$slice: 1`），但会抑制投射元素中的`warehouse`字段。从MongoDB 4.4开始，要获得相同的结果，请使用带两个独立[`$project`](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project)阶段的[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)方法。

**`$`位置运算符和`$slice`限制**

从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) 投射不能包含`$slice`投射表达式作为`$`投射表达式的一部分。例如，从MongoDB 4.4开始，以下操作无效：

  ```js
  db.inventory.find( { "instock.qty": { $gt: 25 } }, { "instock.$": { $slice: 1 } } ) // Invalid starting in 4.4
  ```

MongoDB已经有一个[限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names)，即顶级字段名称不能以美元符号（`$`）开头。在以前的版本中，MongoDB返回`instock`数组中与查询条件匹配的第一个元素（`instock.$`）； 即位置投射`"instock.$"`优先，而`$slice：1`是空操作。`"instock.$"：{$slice：1}`不排除任何其他文档字段。

## 会话

**会话和`$external`用户名限制**

  *在版本3.6.3中更改*：要与`$external`身份验证用户（即Kerberos，LDAP，x.509用户）一起使用会话，用户名不能大于10KB。

**会话空闲超时**
  
在30分钟内未执行任何读或写操作或未使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) 刷新的会话在此阈值之内被标记为已过期，并且MongoDB服务器可以随时将其关闭。关闭会话将终止所有正在进行的操作以及与该会话关联的已打开游标。这包括使用[`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout) 或 [`maxTimeMS()`](https://docs.mongodb.com/manual/reference/method/cursor.maxTimeMS/#mongodb-method-cursor.maxTimeMS) 大于30分钟配置的游标。
  
考虑一个发出[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)命令的应用程序。服务器返回一个游标以及由[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)的 [`cursor.batchSize()`](https://docs.mongodb.com/manual/reference/method/cursor.batchSize/#mongodb-method-cursor.batchSize)定义的一批文档。每次应用程序从服务器请求新一批文档时，会话都会刷新。但是，如果应用程序花费超过30分钟的时间来处理当前批次的文档，则该会话将被标记为已过期并关闭。当应用程序请求下一批文档时，服务器将返回错误，因为在关闭会话时游标已被杀死。
对于返回游标的操作，如果游标可能闲置30分钟以上，请使用[`Mongo.startSession()`](https://docs.mongodb.com/manual/reference/method/Mongo.startSession/#mongodb-method-Mongo.startSession) 在显式会话中发出该操作，并使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) 命令定期刷新该会话。例如： 
  
  ```js
  var session = db.getMongo().startSession()
  var sessionId = session.getSessionId().id
  
  var cursor = session.getDatabase("examples").getCollection("data").find().noCursorTimeout()
  var refreshTimestamp = new Date() // 记录操作开始的时间
  
  while (cursor.hasNext()) {
  
    // 检查距离上一次刷新是否已经过去了5分钟
    if ( (new Date()-refreshTimestamp)/1000 > 300 ) {
      print("refreshing session")
      db.adminCommand({"refreshSessions" : [sessionId]})
      refreshTimestamp = new Date()
    }
  
    // 正常地处理游标
  
  }
  ```
  
  在示例操作中，[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)方法与显式会话相关联。游标使用[`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout)配置，以防止服务器在空闲时关闭游标。`while`循环包含一个代码块，使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions)每5分钟刷新一次会话。由于会话将永远不会超过30分钟的空闲超时，因此游标可以无限期保持打开状态。
  
  对于MongoDB驱动程序，请参考[驱动程序文档](https://docs.mongodb.com/drivers/)中有关创建会话的说明和语法。

## 终端

[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)终端提示符每行的限制为4095个代码点。如果您输入的行中包含4095个以上的代码点，则将被截断。


------

译者：phoenix

时间： 2021.04.26

原文： https://docs.mongodb.com/manual/reference/limits/

版本： 4.4
