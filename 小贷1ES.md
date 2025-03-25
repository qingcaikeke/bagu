https://www.elastic.co/guide/cn/elasticsearch/guide/current/search.html

https://juejin.cn/post/6993148013715652616

[es架构](https://wangdongbing.xyz/2020/06/01/elasticSearch%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/ES%E6%9E%B6%E6%9E%84%E5%8E%9F%E7%90%86/)

[近实时搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-indices.html)（esguide）

#### lucene

文档 —— 分词 ——  字典序排序 —— 倒排索引(磁盘) —— term index 建目录树(内存) —— 查找时二分查找目录树确定磁盘中大概位置，进一步得到文档id —— 找到文档内容本身，返回

文档中字段 —— 建索引 —— 根据字段搜索排序

segment：**倒排索引**用于搜索，**Term Index** 用于加速搜索，**Stored Fields** 用于存放文档的原始信息，以及 **Doc Values** 用于排序和聚合(字段的倒排)

四个内容共同组成一个segment，是一个具备**完整搜索功能的最小单元**。

插入：多个文档生成一份segment，如果新增文档，就要同时更新该seg的多个数据结构，并发性能差，所以规定segment 一旦生成，则不能再被修改。新文档加入就生成新的seg，查的时候并发读多个seg，同时不定期合并多个小 segment，即段合并(不会阻塞前台搜索功能)。

多个seg就可以构成一个单机文本搜索库，也就是lucene

删除：es会创建删除段，删除段包含所有被删除的文档id，搜索会检查文档id是否在删除段中，如果是则忽略。在

段的合并时，es会检查文档id是否在删除段中，如果在，则跳过该文档，不将其包含在新的合并段中。

### ES架构

[mp.weixin.qq.com/s/Ve950zBaOu8VzX9kdqL2Wg](https://mp.weixin.qq.com/s/Ve950zBaOu8VzX9kdqL2Wg)

给lucene做高性能，可扩展，高可用优化，就得到了es

**高性能：**分类，不同index存到不同lucene，搜索时也分类搜索；

​	一个index数据量还是太大，再分shard，每个shard本质上就是一个独立的lucene，把读写操作分摊到多个shard，减少争抢

**可扩展：**分片都在一个机器上，单机 cpu 和内存过高，影响性能。

​	所以把shard分散部署到多个节点，一个node存几个shard，性能不够就加node

**高可用：**shard分主分片和副本分片。主分片挂了，用副本替代。副本也可以分散读压力

读哪个靠负载均衡，主副都行，但写只能往主分片写

请求可以发送到任意节点，每个节点知道集群中任意文档的位置）

**写入：**先找 Coordinate 节点，该节点根据 文档 id 哈希路由，确定文档要存到哪个分片，再找到主分片再哪个node，主分片生成倒排索引等，写入新seg；同时主并行转发到从（保证尽快一致)，得到回复后（可配置），主分片给协调节点回ack，协调节点回客户端

**更新：**找到原文档，更新指定内容，生成新段（段是不可变的），原文档标记为删除。主分片完成本地处理后，把

**变更日志**转发给从(文档的 ID、要更新的字段及其新值、版本号等)，副本执行与主相同的相关操作。

**搜索：**先找协调，协调转发到的分片，并发搜索，协调整合结果

es与kafka架构很类似，es用于分类的消息的`index name`类似kafka的`topic`，分片`shard`类似`partition`分区，es一个node部署若干个shard，kafka一个broker可以存在多个partition分区

**Node分化：**master主节点负责协调集群，创建删除索引，跟踪哪些节点是集群的一部分，决定分片分配到哪个节点

所有节点默认为coordinate节点

ingest node：摄取节点，用于写入es前对数据进行预处理，指定需要的pipline，pipline定义了一系列processor
pipline包括两个主要的参数description和processors

```
{
  "description" : "...",//说明pipeline的功能，主要是一些描述信息。
  "processors" : [ ... ]//是一个list,es按顺序执行，es内置了大量方便的proc：字段数据类型转换(convert),时间戳解析(date),模式匹配提取字段中的数据，储存到指定字段(grok(意会))，删除指定字段(remove),字段重命名(rename),更新字段(set),解析uri
}
```

#### **并发控制：**

每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， `_version` 的值会递增。用以防止一部分修改不会覆盖另一部分。

当进行更新操作时，ES 会检查文档的版本号是否与请求中的版本号一致。如果不一致，说明文档在更新过程中被其他操作修改过，更新请求会失败，需要重新发起请求。

一种常见的结构是使用其他数据库做主库，使用es搜索，这意味着主库数据发生变化，就要将其拷贝到es中，如果多个进程负责同步，就可能出现并发问题，可通过主数据库中的版本字段作为es的外部版本号解决该问题

#### 近实时查询

写入数据的时候，采用内存buff+文件系统缓存+磁盘三级结构，数据大概一秒后能查到(buffer 到osCache的时

间)（index Buffer 是ES内存中的一部分；OS 系统文件缓存是操作系统的，不属于ES内存）

1. 数据会同时写入buffer缓冲区和**translog**日志文件(持久化，断电恢复)

2. **buffer缓冲区**满了或者到时间了（默认1s），执行refresh。refresh会把es缓冲区数据转换成segment并写

   入os cache，数据只有到了系统文件缓存才能被搜索到。同时在os cache中会发生段合并

3. 当translog达到大小的阈值(默认512M)或者flush默认时长（30m），则会执行flush操作：

   内存中数据写入新的segment放入缓存（清空内存区）

   一个commit point写入磁盘，表示哪些segment已写入磁盘

   将缓存的segement写入磁盘（fsync命令）

   清空旧的translog（因为没用了）

4. **translog**日志文件也需要持久化到磁盘：

   同步刷盘：每次修改操作完成后立刻执行fsync命令刷盘

   异步刷盘：默认每5s执行fsync命令刷盘

![image-20250218195225815](C:\Users\16776\AppData\Roaming\Typora\typora-user-images\image-20250218195225815.png)

```
storeKOl
  "settings": {
    "max_result_window": 10000000, //一次查询最多返回1000万个文档
    "refresh_interval": "10s", // 5s把新数据刷新到os cache，意味着新数据要5s才能查到
    "translog": { //日志异步同步，每隔5s把日志从os cache刷到磁盘
      "sync_interval": "5s",
      "durability": "async"
    },
    "analysis": {}
    "number_of_replicas": "1"
  }
```

#### 什么是ES

es是面向文档的，储存整个文档，同时还能索引文档的内容，支持对文档进行索引，检索，排序，过滤

es可以横向扩展至数百（甚至数千）的服务器节点，同时可以处理PB级数据。

天生就是分布式的，并且在设计时屏蔽了分布式的复杂性，如：按集群节点均衡分片，对索引和搜索过程负载均衡，数据冗余防止数据丢失，对任一节点的请求会路由到存有相关数据的节点，无缝整合新节点，重新分配分片

为什么有数据库的情况下还要有es？：传统数据库查文档中含有某些内容效率低，没法查相关性（全文搜索），没复杂搜索、排序

#### **文档元数据**

_ index,文档所在的索引

_ type,文档类型，每个类型有自己的映射，储存在同一索引下

_id,文档唯一标识

补：新的es中，推荐使用单一类型"_doc"来代表文档类型，而不再需要显式指定其他自定义类型。

es自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。

### DSL

query，sort，aggs级别并列，它们都属于搜索参数

```
{ #查询结果分桶,再根据某种规则排序
    "query": {},
    "aggs": {}
    "sort":
}
```



```
GET _search
{
	"query" : { 
		"match":{ 
			"field" : yuanshen 
		}
	}
}
```

query被称为请求参数，match被称为查询语句

aggs与query并列

```
query (bool must/filter/should/must_not) [match,range,term,terms,has_parent]
只有一个条件时，query可直接跟match/range等
多个条件时，query bool {must[b,b,b],should[c,c,c]},只用到了must/should中的一个，也要有bool
bool查询的四种子句：子句间‘，’连接，它们均是一种数组，数组里面是对应的判断条件
	1.需要算分就用must，否则用filter，因为他可以使用缓存
	2.must相当于sql中的and，should相当于or，must_not相当于not
常见查询：
	1.精准查询term，term单值，即字段只有一个值的时候，terms多值，后跟一个数组
	2.匹配查询match，查询的时候会有分析器分析，相当于模糊匹配
```

#### match_phrase

当一个字符串被分析时，分析器不仅只返回一个词条列表，它同时也返回原始字符串的每个词条的位置、或者顺序信息

设置slop值允许一定程度上的插词、倒序等

match_phrase如果不开slop太严格，开了slop，其实就不保证前后次的顺序问题了。有时候前后词颠倒了意思完全变了，比如tom eat jerry和jerry eat tom。此时如果保存tom eat或者eat jerry这两组词的词序，会更贴合意思。所以提供了[相关词shingle](https://www.elastic.co/guide/cn/elasticsearch/guide/current/shingles.html)

- 对于检索词为长句的情况，使用match检索，shingle叠加查询效果更好；

- 对于固定短语检索，直接match_phrase表现更好。当然也可以先match再match_phrase；

```json
#组合查询，叠加计分
比如七个词的检索词，能匹配上六个已经很相关了。但是match_phrase要求必须全都出现。
使用match做初步检索，再使用match_phrase提高精准度：
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must": {
        "match": { 
          "title": {
            "query": "quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      "should": {
        "match_phrase": { 
          "title": {
            "query": "quick brown fox",
            "slop":  50
          }
        }
      }
    }
  }
```

1. 在候选很多的情况下，可以使用match_phrase作为主查询，查出来的结果都相当精准。并适当叠加以下设置：

   1. slop，允许一定程度的乱序；
   2. bool查询组合查standard index，给相同词形加分；


```
（先召回，再叠加别的查询，提高分数）
GET /puzzle/_search
{
  "query": {
      "bool": {
         "must": {
            "match_phrase": {
               "article.reb_eng": {
                  "query": "puzzles & survival",
                  "slop": 1
               }
            }
         },
         "should": [ //单字段多索引的另一个分析器，给相同词形加分
           {
             "match": {
               "article.reb_stan": "puzzles & survival"
            }
           }
         ]
      }
   }
}
```

```
//短语匹配，适合于需要精确匹配特定短语的场景
//先分词，然后检查顺序和临近度，要求字段包含所有短语，同时相对位置不变，可以间隔slop个单词
//"quick fox","quick fox brown"不行
//"quick a brown fox" 可以
{
  "query": {
    "match_phrase": { 
      "tag": {
        "query": "quick brown fox",
        "slop": 1
      }
    }
  }
}
```

#### 打分

 [打分](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/scoring-theory.html)

```
1. 词项频率Term Frequency, TF：一个词项在文档中出现的频率越高，通常意味着文档与查询更相关。
2. 逆文档频率Inverse Document Frequency, IDF：一个词项在整个索引中出现的频率越低，它对文档相关性的贡献越大。
3. 字段长度归一化Field-length Norm：字段越短，通常意味着每个词项对文档相关性的贡献越大。
4. 短语匹配的精确度：match_phrase查询要求词项不仅出现在文档中，而且它们的出现顺序和邻近度也必须与查询短语相同。因此，词项的顺序和它们之间的距离也会影响得分。
```

#### Mapping

默认情况下，文档中的每一个字段都会默认被索引（拥有一个倒排索引），只有这样他们才是可被搜索的

_source字段/\_all字段）代表全文

在mapping中，定义字段的类型及处理方式，保证可以正确储存，索引。

可以根据数据类型对数据进行限制和约束，如数值型给个取值范围，日期指定日期格式

也可以定义并使用自定义的分析器

确定字段是否需要被索引，索引了是否需要储存原始值，是否需要记录位置信息

可以定义嵌套对象（nested）使得 ES 能够有效地索引和查询这些复杂结构的数据。

也可以单字段多索引

###  倒排索引建立

为了更好的匹配，创建倒排索引的分词过程中。词条还需要标准化(如转小写，去除词根，处理特殊字符，处理同义词)，搜索时对搜索文本进行相同的**分析**，就能召回对应文档

#### **分析器**：

分析器 = 字符过滤器 + 分词器 + token过滤器

字符过滤器：去除html，&转and；

分词器：空格标点分割，ngram；

token过滤齐：小写化，去除 a/and/the等，增加词条如jump和leap）

在索引时可以为不同字段指定不同的索引分析器

**测试分析器**

```
用于测试分析器，看文本是如何被分析的api，指定分析器和要分析的文本
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
查看文本在当前字段分析后的结果
GET stored_kol/_analyze
{
  "field":"keyword",
  "text": "yuansHan"
}
```

#### **实习用到的分析器**

标准分析器，英语分析器（去除英语无用词，去除词根提取词干），自定义分析器(ngram自动补全)

autocomplete_sentence(lowercase+sentence_edge_ngram) + reb_eng , keyword

```
- autocomplete_sentence：自定义的ngram analyzer，将整个句子按字符切分，切分出的字符串小写；
- autocomplete_sentence_search：自定义的搜索时用的analyzer，讲整个句子小写，作为整体用于搜索；
```

```
搜索3.2中，关键词联想中，如果超过20个字符只取前20，进行mathPhase，不应该在应用代码中手动截断tag.substring(0,Math.min(tag.length(), 20))
而应该在索引时为该字段的搜索分析器（search analyzer）添加一个 truncate 令牌过滤器（token filter），令牌过滤器是分析器的一部分，可以对分词器（tokenizer）生成的词项进行进一步的处理。例如，它可以移除停用词、转换大小写或截断词项
性能优化：在数据库层面处理截断可以减少应用层的计算负担，提高查询性能。
一致性：在索引时应用截断规则可以确保所有相关的查询都遵循相同的长度限制，避免因应用代码中的差异导致的不一致性。
易于维护：将截断逻辑放在 Elasticsearch 的映射（mapping）中，可以使得维护和更新截断规则更加集中和简单。
```

其他常用分析器

IK分词器：默认的分词器是把每个字看成一个词，这显然有问题，把中文内容划分成关键字。ik分词器提供两个分词算法（analyzer：ik_smart粗粒度分词，ik_max_word细粒度）可以从配置文件编辑自定义词库

#### 为什么kol和media不用嵌套文档，而用父子文档

为什么不选nested文档(嵌套文档)：

1. 无法做到独立更新，因为他们实际是一条文档；
2. media需要经常更新，每次都要重建该kol所有的kol+media大文档；

为什么选择父子文档：

1. kol和media单独存储，互不影响；
2. 均可独立更新；
3. media更新频繁；
4. 一对多，尤其是子文档远多于父文档的时候，很适合父子型文档存储。

相关方法

父子型一对多关系：https://www.elastic.co/guide/cn/elasticsearch/guide/current/parent-child.html

### 查询优化

[es调优实战](https://zhuanlan.zhihu.com/p/45331117)

[es调优2](https://mp.weixin.qq.com/s/4exhftt-9Xqb8EZIujJaLA)

##### 字段类型选择

- 数据类型选型；
- 是否需要检索；
- 是否需要排序+聚合分析；
- 是否需要另行存储。

1. es支持4种数字类型：byte，short，integer，long。如果最小的类型就合适，那么就用最小的类型，节省磁盘空间。
2. 不需要做全文检索的字段使用 keyword类型代替text类型，这样可以避免在建立索引前对这些文本进行分词。

##### 使用自动id

写入数据不指定_id，让ES自动产生，业务上的主键ID可以当做一个Field

当用户显示指定id写入数据时，ES会先发起查询来确定index中是否已经有相同id的doc存在，若有则先删除原有doc再写入新doc。这样每次写入时，ES都会耗费一定的资源做查询。如果用户写入数据时不指定doc，ES则通过内部算法产生一个随机的_id，并且保证_id的唯一性，这样就可以跳过前面查询_id的步骤，提高写入效率。

##### **深分页**

因为es索引是分片储存的，假设分5片，结果要前10，就需要每个分片取前10，召回50个，然后这50个再排序

假设现在要第1000页（结果10001-10010），就需要每个分片召回10010个，然后五万多个排序取10个

避免深度分页查询建议使用 Scroll 进行分页查询(滚动分页)。

正常查10000到20000，要再搜一次

scroll可以直接获取后续结果

scorll有个有效期，有效期内可以根据上一次id直接返回10000到20000

##### **query-bool-filter取代普通query**

默认情况下，ES通过一定的算法计算返回的每条数据与查询语句的相关度，并通过score字段来表征。
但对于非全文索引的使用场景，用户并不care查询结果与查询条件的相关度，只是想精确的查找目标数据。
此时，可以通过query-bool-filter组合来让ES不计算score，并且尽可能的缓存filter的结果集，

即使是term精准匹配/keyword类型的字段也会算分，会考虑词频逆文档频率等

- 使用`term`查询时，查询输入不变，去和字段分析后的内容进行精确匹配。
- 使用`match`查询时，查询输入和字段内容都会被分词，然后在分词结果中进行匹配。

```
# 普通查询
  "query": {
    "term" : { "user" : "Kimchy" }    
# query-bool-filter 加速查询
  "query": {
    "bool": {
      "filter": {
        "term": { "user": "Kimchy" }
```

##### 使用routing

对于数据量较大的index，一般会配置多个shard来分摊压力。这种场景下，一个查询会同时搜索所有的shard，
然后再将各个shard的结果合并后，返回给用户。对于高并发的小查询场景，每个分片通常仅抓取极少量数据，
此时查询过程中的调度开销远大于实际读取数据的开销，且查询速度取决于最慢的一个分片。

开启routing功能后，ES会将routing相同的数据写入到同一个分片中
(也可以是多个，由index.routing_partition_size参数控制)。
如果查询时指定routing，那么ES只会查询routing指向的那个分片，可显著降低调度开销，提升查询效率

##### 结构化搜索

查35岁以上的员工

#### **es面试，拼写矫正，同义词等**

面试官问这个问题是想了解你对于Elasticsearch（ES）的优化经验以及如何处理搜索中的常见问题，比如拼写错误、同义词、短语搜索等。以下是一些可能的优化措施和解决方案：

1. **拼写校正**：
   - 使用`ngram`或`edge_ngram`分析器来索引词汇的子串，这样即使用户输入的查询包含拼写错误，也能匹配到正确的词汇。
   - 使用`hunspell`插件或`completion suggester`来提供拼写建议。
2. **同义词处理**：
   - 创建同义词字典，在索引和搜索时使用同义词过滤器，这样即使用户使用同义词搜索，也能找到相关的结果。
3. **短语搜索**：
   - 使用`match_phrase`查询来处理短语搜索，并调整`slop`参数来允许单词之间有一定程度的灵活性。
4. **性能优化**：
   - 使用过滤器（filter）而不是查询（query）来执行不依赖于评分（score）的搜索，因为过滤器可以被缓存。
   - 选择合适的字段数据类型，比如使用`keyword`类型进行聚合和排序，使用`text`类型进行全文搜索。
   - 优化查询DSL，避免使用复杂的查询，减少网络开销。
5. **索引优化**：
   - 定期进行索引优化，比如合并分片、删除旧数据、优化映射等。
   - 根据数据量和查询模式选择合适的分片和副本数量。
6. **硬件和配置优化**：
   - 根据ES的建议配置JVM参数。
   - 确保有足够的硬件资源，比如CPU、内存和I/O性能。
7. **查询分析**：
   - 使用`explain`参数来分析查询的执行计划，了解查询的性能瓶颈。
   - 使用`profile` API来获取查询的性能分析报告。
8. **缓存利用**：
   - 合理利用ES的缓存机制，比如fielddata缓存和过滤器缓存。
9. **监控和告警**：
   - 使用工具如Grafana等来监控集群状态和性能指标。
   - 设置告警机制，以便在性能下降或出现异常时及时响应。

### 具体查询语句

##### 文档信息

`_source` 字段：原始文档

GET /index/_doc/id

```sense
GET stored_kol/_doc/321760/_source //只取原文档，不要元数据
GET stored_kol/_doc/321760/?_source=avg_watch,platform //只取需要的字段
```

get字段名

```
GET keywords_tags 结果有mappings和settings
GET keywords_tags/_mapping 了解数据模型
GET keywords_tags/_settings 分片数，副本数等
```

##### 搜索语句

```
{
  "query": {
    "bool": {
      "must": [//算分
        {
          "range": {
            "time": {
              "gte": 10,
              "lte": 20
            }
          }
        },
        {
          "term": { //精确匹配，不适用于text类型，因为查询语句不会分词，而字段被分词，这就导致查不到结果
            "age": {
              "value": "VALUE"
            }
          }
        }
      ], 
      "filter": [
        {
          "term": {
            "platform": "youtube"
          }
        }
      ]
    }
  }
}
```

##### 搜索结果

```sense
GET /index/_search
{
  "took" : 4, //查询耗时ms
  "timed_out" : false, //是否超时,可以在查询中指定
  "_shards" : { //多少分片参与查询
    "total" : 15,
    "successful" : 15,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : { //匹配到的文档总数
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 1.0,//匹配到的文档中最高得分
    "hits" : [ //查询结果的前十个文档
      {
        "_index":   "us",
        "_type":    "tweet",
        "_id":      "7",
        "_score":   1,
        "_source":  { //原文档
        }
      },
      {
      }
    ]
  }
}
```

##### 添加字段

```
PUT stored_kol/_mapping  //针对索引映射进行设置
{
  "properties": {
    "similarity": { // 新增加一个字段，名为similarity
      "type": "dense_vector", // 字段类型为稠密向量
      "dims": 768 // 维度为768维
      "analyzer": "standard", // 可选，指定分析器，vector字段没有
      "fields": { // 可选，单字段多索引
            "xx.yy": {
                "type": "keyword",
                "analyzer": "kk"
            }
      }
    }
  }
} 
```

修改映射后，新插入的文档会包含新字段，但已有文档不会自动添加该字段。如果你需要更新已有文档，为其添加新字段

```
POST stored_kol/_update_by_query
{
    "script": {
        "source": "ctx._source.new_field = 'default_value'",
        "lang": "painless"
    },
    "query": {
        "match_all": {}
    }
}
```

- `POST stored_kol/_update_by_query` 表示要对 `stored_kol` 索引中的文档进行批量更新。
- `script` 部分定义了要执行的脚本，这里将 `new_field` 字段的值设置为 `default_value`。
- `query` 部分定义了要更新的文档范围，这里使用 `match_all` 查询表示更新所有文档。

##### 分桶

```
GET stored_kol/_search
{
  "aggs": {
    "NAME": {
      "AGG_TYPE": {}
      "aggregations": {} #聚合后每个桶的字段，内部可以再次聚合，也可以使用script得到
    }
  }
}
AGG_TYPE：
  桶聚合：terms，range，filters(根据指定范围分桶)，histogram(根据数值范围分桶，直方图聚合),nested
  指标聚合:avg,sum,min,max
```

