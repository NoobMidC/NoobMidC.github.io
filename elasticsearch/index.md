# Elasticsearch


<!--more-->



# ElasticSearch

​		 Elasticsearch是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。



## ES的特点和优势

1. 分布式实时文件存储，可将每一个字段存入索引，使其可以被检索到。

   > 分布式：索引分拆成多个分片，每个分片可有零个或多个副本。集群中的每个数据节点都可承载一个或多个分片，并且协调和处理各种操作；负载再平衡和路由在大多数情况下自动完成。

2. 实时分析的分布式搜索引擎。

3. 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据;也可以运行在单台PC上;

4. 支持插件机制，分词插件、同步插件、Hadoop插件、可视化插件等。



## ES倒排索引

​		倒排索引是首先知道了每个关键词都出现在了哪些文档里，**从关键词搜文档（关键词→文档）**

### 倒排索引原理	

​		倒排索引主要由单词词典（Term Dictionary）和倒排列表（Posting List）及倒排文件(Inverted File)组成。

- **单词词典（Term Dictionary）：**搜索引擎的通常索引单位是单词，单词词典是由文档集合中出现过的所有单词构成的字符串集合，单词词典内每条索引项记载单词本身的一些信息以及指向“倒排列表”的指针。
- **倒排列表(PostingList)：**倒排列表记载了出现过某个单词的所有文档的文档列表及单词在该文档中出现的位置信息及频率（作关联性算分），每条记录称为一个倒排项(Posting)。根据倒排列表，即可获知哪些文档包含某个单词。
- **倒排文件(Inverted File)：**所有单词的倒排列表往往顺序地存储在磁盘的某个文件里，这个文件即被称之为倒排文件，倒排文件是存储倒排索引的物理文件。

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/watermark%252Ctype_ZmFuZ3poZW5naGVpdGk%252Cshadow_10%252Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1emhlbnNtYXJ0%252Csize_16%252Ccolor_FFFFFF%252Ct_70.png)

### 字典树

​		Trie树是一种前缀树，我们之前也有介绍过，一般应用在快速查询中，例如搜索提示，当你输入前半部分，会提示后半部分的内容。字典树用一句话表示就是根据字符串的前缀构成的树结构。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高

​		Trie 的核心思想是空间换时间，利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。它有 3 个基本性质：

- 根节点不包含字符，除根节点外每一个节点都只包含一个字符
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
- 每个节点的所有子节点包含的字符都不相同

​		对于中文的字典树，每个节点的子节点用一个哈希表存储，这样就不用浪费太大的空间，而且查询速度上可以保留哈希的复杂度 O(1)。

> 搜索字典项目的方法为：(来自百度百科)
>
> 1. 从根结点开始一次搜索；
> 2. 取得要查找关键词的第一个字母，并根据该字母选择对应的子树并转到该子树继续进行检索；
> 3. 在相应的子树上，取得要查找关键词的第二个字母,并进一步选择对应的子树进行检索。
> 4.  迭代过程……(重复123)
> 5. 在某个结点处，关键词的所有字母已被取出，则读取附在该结点上的信息，即完成查找。

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-2989bd84148a89b1eb44f622bc975292_1440w.jpg)

> 应用实例: 1. 字符串的快速检索;2. 字符串排序; 3. 最长公共前缀; 4, 搜索引擎



### 不变的倒排索引

​		早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的修改变化便可以被检索到。

不变性的优点: 

1. 不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题
2. 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升;
3. 其它缓存(像 filter 缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化;
4. 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。

**缺点**:   当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制;



### 动态更新的倒排索引

​		为了保持倒排索引的不变性同时实现更新,  通过增加新的补充索引来反映最近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始查询完后再对结果进行合并（因为不重写索引，所以旧索引要合并减少空间大小）。

​		Elasticsearch 基于 Lucene, 其中有按段搜索的概念; 每一段本身都是一个倒排索引，但索引在 Lucene 中除表示所有段的集合外，还增加了提交点的概念：一个列出了所有已知段的文件。![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.1ca78zo9jedc-20220903142211189.webp)

- 按段搜索

  1. 新文档被收集到内存索引缓存

     ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.2rwt32k2s6a0.webp)

  2. 不时地，缓存被提交: 一个新的段,即一个追加的倒排索引被写入磁盘; 一个新的包含新段名字的「提交点」 被写入磁盘; 磁盘进行同步，所有在文件系统缓存中等待的写入都刷新到磁盘，以确保它们被写入物理文件.

  3. 新的段被开启，让它包含的文档可见以被搜索

  4. 内存缓存被清空，等待接收新的文档

  ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.4jiajnwyaum0.webp)

- 按段搜索的好处

  1. 当一个查询被触发，所有已知的段按顺序被查询。词项统计会对所有段的结果进行聚合，以保证每个词和每个文档的关联都被准确计算。 这种方式可以用相对较低的成本将新文档添加到索引。
  2. 段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档的更新。取而代之的是，每个提交点会包含一个 .del 文件，文件中会列出这些被删除文档的段信息。当一个文档被」删除」时，它实际上只是在 .del 文件中被「标记」删除。一个被标记删除的文档仍然可以被查询匹配到，**但它会在最终结果被返回前从结果集中移除**。
  3. 文档更新也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被检索到一个新的段中。可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。



### ES索引文档的过程

1. 请求发送到Coordinating Node，如果该节点不是Master节点，需要将该请求转发到Master.Master节点通过路由算法，确定该分片在哪个节点上：

   ​		shard = hash(_routing) % (num_of_primary_shards)默认使用文档ID作为_routing值，也可以通过API指定_routing值

2. 分片节点收到请求后，会将请求写入到Memory Buffer，然后定时（默认是每隔1秒）写入到Filesystem Cache(磁盘缓存)，这个从Momery Buffer到Filesystem Cache（磁盘缓存）的过程就叫做**Refresh**。在写入Buffer的同时，会同时写一个Transaction Log(每个分片会有一个Log文件)。当做refresh时，Buffer会被清空，Transaction Log不会清空。

3. ES每30分钟，会有一个Flush操作。该操作会调用fsync，将Filesystem Cache中的数据写入segment文件，旧的translog将被删除并开始一个新的translog

4. ES会自动进行Merge操作，该操作会将多个Segment文件合并，在.del文件中被标记为删除的文档将不会被写入Segment，并且清空.del文件中的内容



### ES Refresh过程(近乎实时搜索)

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/webp-20220903143538596)

- 创建索引时，数据并不会直接写入Segment文件，而是会先写入一个Index Buffer缓冲区。而将Index Buffer中数据写入Filesystem Cache（磁盘缓存)的过程就叫Refresh.

  ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.1i72wek5rb1c.webp)

- Refresh的频率默认是每秒一次。可通过index.refresh_interval进行配置。Refresh后，数据就可被搜索到了，这也是为什么ES被称为近实时搜索。

- Index Buffer被占满时，也会触发Refresh，默认值是JVM的10%



### Transaction Log

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/webp-20220903143819158)

​		Segment写入磁盘的过程相对耗时，所以借助文件系统缓存，Refresh时，先将Segment写入文件缓存中，以开放查询。但为了保证数据不会丢失，所以在创建索引时，会同时写Tansaction Log，类似操作日志。在ES进行Refresh时，Index Buffer会被清空，Transaction Log不会清空。

> translog 提供所有还没有被刷到磁盘的操作的一个持久化纪录。当 Elasticsearch 启动的时候，它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 translog 中所有在最后一次提交后发生的变更操作。
>
> translog 也被用来提供实时 CRUD。当你试着通过 ID 查询、更新、删除一个文档，它会在尝试从相应的段中检索之前，首先检查 translog 任何最近的变更。这意味着它总是能够实时地获取到文档的最新版本。
>
> 默认 translog 是每 5 秒被 fsync 刷新到硬盘， 或者在每次写请求完成之后执行(e.g., index, delete, update, bulk)。这个过程在主分片和复制分片都会发生。

### ES Flush

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/webp-20220903143944081)

Flush操作：调用refresh，清空index buffer; 清空transaction log

Flush触发条件：1. 默认30分钟调用一次; 2. Transaction Log 默认达到512M



### 持久化过程

![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.2wqzo0d8lye0.webp)

1. 一个文档被索引之后，就会被添加到内存缓冲区，并且追加到了 translog 日志

   ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.4cy2su25u8o0.webp)

2. 刷新（refresh）使分片每秒被刷新（refresh）一次：

   - 这些在内存缓冲区的文档被写入到一个新的段(文件缓冲区)中，且没有进行 fsync 操作;
   - 这个段被打开，使其可被搜索;内存缓冲区被清空![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.3ylja6rewxk0.webp)

3. 这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志

   ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.4czulsx2el00.webp)

4. 每隔一段时间30min或者 translog 变得足够大，索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行;

   - 所有在内存缓冲区的文档都被写入一个新的段;缓冲区被清空
   - 一个提交点被写入硬盘
   - 文件系统缓存通过 fsync 被刷新（flush）;老的 translog 被删除

   ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.2n6d7ac5vze0.webp)



### 段合并

​		每一个段都会消耗文件句柄、内存和 cpu 运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。

​		Elasticsearch 通过在后台进行段合并来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。段合并的时候会将那些旧的已删除文档从文件系统中清除。被删除的文档（或被更新文档的 旧版本）不会被拷贝到新的大段中。

- 当检索的时候，刷新（refresh）操作会创建新的段并将段打开以供搜索使用

- 合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中。这并不会中断检索和搜索、

  ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.7azx5r0frvo0.webp)

- 一旦合并结束，老的段被删除

  1. 新的段被刷新（flush）到了磁盘。 写入一个包含新段且排除旧的和较小的段的新提交点

  2. 新的段被打开用来搜索; 老的段被删除

  ![image](https://raw.githubusercontent.com/NoobMidC/pics/main/image.rzljm90yps0.webp)

  > 合并大的段需要消耗大量的 I/O 和 CPU 资源，如果任其发展会影响搜索性能。Elasticsearch 在默认情况下会对合并流程进行资源限制，所以搜索仍然有足够的资源很好地执行。



## 查写删的流程

​		以下均针对集群而言

### ES写数据的流程

1. 客户端发送写请求到某个节点，此节点为协调节点. 其将请求通过路由计算，将请求发到指定的节点。

   - 写请求是写入 primary shard，然后同步给所有的 replica shard；

     > 当主分片把更改转发到副本分片时，它不会转发更新请求。相反，它转发完整文档的新版本。请记住，这些数据更改文档将会异步转发到副本分片，并且不能保证数据更改文档以发送它们相同的顺序到达。如果 Elasticsearch 仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

   - 读请求可以从 primary shard 或 replica shard 读取，采用的是随机轮询算法。

![截屏2022-09-03 下午12.24.33](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%8812.24.33.png)

2. 数据到达指定节点后, 对应的primary shard将数据先写入内存 buffer，然后每隔 1s，将数据 refresh 到 os cache，到了 os cache 数据就能被搜索到（所以我们才说 es 从写入到能被搜索到，中间有 1s 的延迟）。每隔 5s，将数据写入 translog 文件（这样如果机器宕机，内存数据全没，最多会有 5s 的数据丢失），translog 大到一定程度，或者默认每隔 30mins，会触发 commit 操作，将缓冲区的数据都 flush 到 segment file 磁盘文件中。

3. 数据写入 segment file 之后(从buffer 刷新到os cache)，同时就建立好了倒排索引。

   ![截屏2022-09-03 下午12.27.00](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%8812.27.00.png)



### ES查数据的流程

> ​	找到所有匹配的结果是查询的第一步，来自多个shard上的数据集在分页返回到客户端的之前会被合并到一个排序后的list列表，由于需要经过一步取top N的操作，所以search需要进过两个阶段才能完成，分别是query和fetch。
>
> ​	流程可以总结为先广播请求到各个分片，然后各个分片取topN，最后再进行整合。 整体流程如下

![1](https://raw.githubusercontent.com/NoobMidC/pics/main/watermark%252Ctype_ZmFuZ3poZW5naGVpdGk%252Cshadow_10%252Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ljY2NzZG4%253D%252Csize_16%252Ccolor_FFFFFF%252Ct_70.png)

1. query阶段: 根据查询获取具体的数据的id

   - 客户端发送search请求到某个node,即协调节点(如node3)

   - 协调节点将查询请求转发到索引的每个主分片或副分片中。

   - 每个分片在本地执行查询，并使用本地的Term/Docuemnt Frequency信息进行打分，添加结果到大小为from+size的本地优先队列中。(在搜索的时候是会查询 Filesystem Cache 的，但是有部分数据还在 Memory Buffer，所以搜索是近实时的)

   - 每个分片返回各自优先队列中所有文档的ID和排序值给协调节点，协调节点合并这些值到自己的优先队列中，产生一个有序全局列表。

   > 查询阶段并不会对搜索请求的内容进行解析，无论搜索什么内容，只看本次搜索需要命中哪些shard，然后针对每个特定shard选择一个副本，转发搜索请求。

   ![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-1289ee073a8f14b5aa57c3e715c685df_1440w.jpg)

   

2. Fetch阶段: 根据query过程的获取的id拿出具体的数据

   - 协调节点对query阶段的数据重新截取数据后，获取到真正需要返回的数据的 id
   - 协调节点向相关NODE发送GET请求
   - 分片所在节点向协调节点返回数据。
   - 协调节点等待所有文档被取得，然后返回给客户端。

   > 在搜索的时候是会查询Filesystem Cache的，但是有部分数据还在Memory Buffer，所以搜索是近实时的。

   ![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-1154c0210c32adc80122669d85f6f17e_1440w.jpg)

   

### ES删数据的流程

- 更新操作就是将原来的数据标识为deleted状态，重新写入数据的过程![截屏2022-09-03 下午2.10.33](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%882.10.33.png)
- 删除操作，commit 的时候会生成一个 .del 文件，里面将某个 doc 标识为 deleted 状态，那么搜索的时候根据 .del 文件就知道这个 doc 是否被删除了![截屏2022-09-03 下午12.51.20](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%8812.51.20.png)





### 批量操作

#### mget(批量读)

​		用单个 `mget` 请求取回多个文档所需的步骤顺序:

1. 客户端向 Node 1 发送 mget 请求
2. Node 1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复，Node 1 构建响应并将其返回给客户端



### bulk API(批量增删改)

​		`bulk API` 允许在单个批量请求中执行多个创建、索引、删除和更新请求; `bulk API` 按执行步骤顺序：

1. 客户端向 Node 1 发送 `bulk` 请求
2. Node 1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。



## ES集群

### 路由计算

```sh
shard = hash(routing) % number_of_primary_shards
位置 = hash(路由哈希值) $ 分片总数
```

- routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards（主分片的数量）后得到余数。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。
- 这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。
- 不带 routing 查询时, 协调节点会将查询请求分发到每个分片上,然后聚合每个分片上的查询结果返回给客户端



### 并发控制

#### 乐观版本控制

​		当我们之前使用 index（索引）的 GET 和 DELETE 请求时，可以通过返回结果看到每个文档都有一个 _version（版本号），当文档被修改时版本号递增。Elasticsearch 使用这个 _version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略掉，也就是不允许执行。

​		我们可以利用 version 号来确保应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。如果该版本不是当前版本号，我们的请求将会失败。

```go
// 老的版本 ES 在写操作时可以指定版本
http://127.0.1:9200/shopping/_update/1001?version=2

// 新版 url 不能直接操作 _version，如果想操作 _version，只能操作由 _version 衍生出来的 _if_seq_no 和 _if_primary_term
http://127.0.1:9200/shopping/_update/1001?if_seq_no=2&if_primary_term=2
```

> 外部系统版本控制:
>
> ​		一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索，这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。
>
> ​		如果你的主数据库已经有了版本号或一个能作为版本号的字段值比如时间戳 timestamp，那么你就可以在 Elasticsearch 中通过增加 `version_type=external` 到查询字符串的方式重用这些相同的版本号，





### 分片策略

1. 分片消耗的资源:

- 一个分片的底层即为一个 Lucene 索引，会消耗一定文件句柄、内存、以及 CPU 运转
- 每一个搜索请求都需要命中索引中的每一个分片，如果每一个分片都处于不同的节点还好，但如果多个分片都需要在同一个节点上竞争使用相同的资源就有些糟糕了
- 用于计算相关度的词项统计信息是基于分片的。如果有许多分片，每一个都只有很少的数据会导致很低的相关度

2. 延迟分片分配机制

      对于节点瞬时中断的问题，默认情况，集群会等待一分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。这样就可以减少 ES 在自动再平衡可用分片时所带来的极大开销。

      修改参数 `delayed_timeout`，可以延长再均衡的时间

### ES集群扩张原理

​		ElasticSearch的节点启动后，它会利用多播(multicast)(或者单播，如果用户更改了配置)寻找集群中的其它节点，并与之建立连接。这个过程如下图所示：

![截屏2022-09-03 下午4.00.40](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%884.00.40.png)



### 集群选举

三种状态

- Leader：接受客户端请求，并向Follower同步请求日志，当日志同步到大多数节点上后告诉Follower提交日志。

- Follower：接受并持久化Leader同步的日志，在Leader告之日志可以提交之后，提交日志。

- Candidate：Leader选举过程中的临时角色。

1. Elasticsearch 的选主是 ZenDiscovery 模块负责的，主要包含 Ping（节点之间通过这个 RPC 来发现彼此）和 Unicast（单播模块包含一个主机列表以控制哪些节点需要 ping 通）这两部分
2. Follower只响应其他服务器的请求。如果Follower超时没有收到Leader的消息，它会成为一个Candidate并且开始一次Leader选举。
3. 对所有可以成为 master 的节点（node.master: true）根据 nodeId 字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第 0 位）节点，暂且认为它是 master 节点
4. 如果对某个节点的投票数达到一定的值（可以成为 master 节点数 n/2+1）并且该节点自己也选举自己，那这个节点就是 master。否则重新选举一直到满足上述条件

> master 节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data 节点可以关闭 http 功能; 选举使用的Raft算法



### 脑裂问题

#### 脑裂问题可能的成因

1. 网络问题：集群间的网络延迟导致一些节点访问不到 master，认为 master 挂掉了从而选举出新的 master，并对 master 上的分片和副本标红，分配新的主分片
2. 节点负载：主节点的角色既为 master 又为 data，访问量较大时可能会导致 ES 停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点
3. 内存回收：data 节点上的 ES 进程占用的内存较大，引发 JVM 的大规模内存回收，造成 ES 进程失去响应

#### 脑裂问题解决方案

1. 减少误判：discovery.zen.ping_timeout 节点状态的响应时间，默认为 3s，可以适当调大

2. 选举触发: 修改参数为1，discovery.zen.minimum_master_nodes:1

   ​		该参数是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个数大于等于该参数的值，且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为（n/2）+1，n 为主节点个数（即有资格成为主节点的节点个数）

3. 角色分离：即 master 节点与 data 节点分离，限制角色



## 优化策略

### 写入速度优化

1. 加大 Translog Flush 间隔， 降低写阻塞(Writeblock)
2. 增加 Index Refresh 间隔，目的是减少 Segment Merge 的次数;由于 Lucene 段合并的计算量庞大，会消耗大量的 I/O，所以 ES 默认采用较保守的策略，让后台定期进行段合并。
3. 批量数据提交
4. 优化存储设备: ES 是一种密集使用磁盘的应用，在段合并的时候会频繁操作磁盘，所以对磁盘要求较高，当磁盘速度提升之后，集群的整体性能会大幅度提高
5. 减少副本数量:写索引时，需要把写入的数据都同步到副本节点，副本节点越多，写索引的效率就越慢;如果我们需要大批量进行写入操作，可以先禁止 Replica 复制，设置 `index.number_of_replicas: 0` 关闭副本。在写入完成后，Replica 修改回正常的状态。



### GC的注意事项

1. 倒排词典的索引需要常驻内存，无法 GC，需要监控 data node 上 segment memory 增长趋势
2. 各类缓存，field cache, filter cache, indexing cache, bulk queue 等等，要设置合理的大小，并且要应该根据最坏的情况来看 heap 是否够用，也就是各类缓存全部占满的时候，还有 heap 空间可以分配给其他任务吗？避免采用 clear cache 等「自欺欺人」的方式来释放内存
3. 避免返回大量结果集的搜索与聚合。确实需要大量拉取数据的场景，可以采用 scan & scroll api 来实现





## Elasticsearch对于大数据量（上亿量级）的聚合如何实现？

​		Elasticsearch 提供的首个近似聚合是 cardinality 度量。它提供一个字段的基数，即该字段的 distinct 或者 unique 值的数目。它是基于 HLL 算法的。(和Redis中的HLL一样)HLL 会先对我们的输入作哈希运算，然后根据哈希运算的结果中的 bits 做概率估算从而得到基数。其特点是：可配置的精度，用来控制内存的使用（更精确 ＝ 更多内存）；小的数据集精度是非常高的；我们可以通过配置参数，来设置去重需要的固定内存使用量。



## 在并发情况下，Elasticsearch如果保证读写一致？

- 可以通过版本号使用乐观并发控制，以确保新版本不会被旧版本覆盖，由应用层来处理具体的冲突
- 另外对于写操作，一致性级别支持 quorum/one/all，默认为 quorum，即只有当大多数分片可用时才允许写操作。但即使大多数可用，也可能存在因为网络等原因导致写入副本失败，这样该副本被认为故障，分片将会在一个不同的节点上重建
- 对于读操作，可以设置 replication 为 sync(默认)，这使得操作在主分片和副本分片都完成后才会返回；如果设置 replication 为 async 时，也可以通过设置搜索请求参数 _preference 为 primary 来查询主分片，确保文档是最新版本



## ES查询为什么存在误差

​		ES执行查询计算分数的时候会包括已删除的文档，这样在进行topN排序时会不准确。



## ES中的scroll查询和普通分页对比

1. size+from的翻页方式，这种翻页方式存在很大的性能瓶颈，时间复杂度O(n)，空间复杂度O(n)。
   - query阶段: 每次查询都会将转发请求给所有分片, 所有分片返回size*from个数据的id和score给协调节点进行排序;
   - fetch阶段: 丢弃无用的数据id,只留下想要的size个数据,然后获取所有文档数据,返回客户端

2. SearchAfter ;Search接口另一种翻页方式是SearchAfter，时间复杂度O(n)，空间复杂度O(1)。

   ​       SearchAfter是一种动态指针的技术，每次查询都会携带上一次的排序值，这样下次取结果只需要从上次的位点继续扫数据

   > 第一次和size+from的方式一样, 后续查询携带上次查询结果末尾的排序条件,减少命中数据

3. SeachScroll;   时间复杂度O(1)，空间复杂度O(1)

   - search阶段时: query阶段和size+from一样;   然后将所有排序后的数据存放在缓存队列中, 取前size个进入fetch阶段, fetch返回后,抛弃前size个文档id  (类似于打了个快照)

   - scroll阶段时: 直接从缓存中取id,  然后进入fetch阶段.

> 在scroll查询时, 都会设置有效时间, 过了有效时间这个缓存就被清理了; 而且es也做了很多压缩处理,大量的ID可以被很好的存在缓存中;





## X-Pack for Elasticsearch的功能和重要性

​		X-Pack 是与 Elasticsearch 一起安装的扩展程序。

​		X-Pack 的各种功能包括安全性(基于⻆色的访问，特权/权限，⻆色和用户安 全性)，监视，报告，警报等。



## Elasticsearch 在部署时，对 **Linux** 的设置有哪些优化方法?

- 关闭缓存 swap;
-  堆内存设置为:Min(节点内存/2, 32GB);
-  设置最大文件句柄数;
-  线程池+队列大小根据业务需要做调整;
-  磁盘存储 raid 方式——存储有条件使用 RAID10，增加单节点性能以及避 免单节点存储故障。

## ES和Mysql对比

![截屏2022-09-03 下午4.17.27](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-03%20%E4%B8%8B%E5%8D%884.17.27.png)

​						

1. 基于分词后的全文检索

​			es分词后，每个字都可以利用FST高速找到倒排索引的位置，并迅速获取文档id列表

2. 精确检索

​			mysql的非聚合索引用上了覆盖索引，无需回表，则速度可能更快

​			es还是通过FST找到倒排索引的位置并获取文档id列表，再根据文档id获取文档并根据相关度算分进行排序，但es还有个杀手锏，即天然的分布式使得在大数据量面前可以通过分片降低每个分片的检索规模，并且可以并行检索提升效率

​		用filter时更是可以直接跳过检索直接走缓存



3. 聚合查询对比

​				mysql做例子，如果没有建立索引，我们需要全遍历一份，到内存进行排序，如果有索引，会在索引树上进行进行范围查询（因为索引是排序了的）

​				es中，如果是做排序，lucene会查询出所有的文档集合的排序字段，然后再次构建出一个排序好的文档集合。es是面向海量数据的，这样一来内存爆掉的可能性是十分大的



总结：ElasticSearch为了提高检索性能，无所不用的压缩数据，减少磁盘的随机读，以及及其苛刻的使用内存，使用各种快速的搜索算法等手段

追求更快的检索算法 — 倒排，二分

更小的数据传输--- 压缩数据（FST，ROF，公共子前缀压缩）

更少的磁盘读取--- 顺序读（DocValue顺序读）

