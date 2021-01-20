# 7.10新功能

以下是Elasticsearch 7.10更新和改进的亮点!有关此发行版的详细信息，请参阅[发行说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/release-notes-7.10.2.html) 和[重大更改](https://www.elastic.co/guide/en/elasticsearch/reference/current/migrating-7.10.html#breaking-changes-7.10)。



###  索引速度提高了

Elasticsearch 7.10提高了20%的索引速度。我们减少了将条目添加到事务日志所需的协调。这种减少允许更多的并发性，并将事务日志缓冲区大小从8KB增加到1MB。但是，对于全文搜索和其他分析密集的用例，性能提高较低。索引链越重，收益就越低，因此涉及许多字段、提取管道或全文本索引的索引链将获得较低的收益。



### 更节省空间的指标

Elasticsearch 7.10依赖于Apache Lucene 8.7，它引入了更高的存储字段压缩，索引的一部分主要存储_source。在我们进行基准测试的各种数据集上，我们注意到空间缩减在0%到10%之间。这种更改对跨文档具有大量冗余数据的数据集特别有帮助，这通常是由我们的可观察性解决方案生成的文档的情况，这些解决方案重复关于在每个文档上生成数据的主机的元数据。



Elasticsearch提供了配置索引的能力。编解码器设置告诉Elasticsearch如何积极地压缩存储的字段。支持的默认值和best_compression都将通过此更改获得更好的压缩。



### 数据层

在Elasticsearch中引入了形式化数据层的概念。数据层是一种简单的集成方法，使用户可以控制优化成本，性能和数据的广度/深度。在此形式化之前，许多用户使用自定义节点属性配置自己的层拓扑，并使用ILM管理集群中数据的生命周期和位置。



通过这种形式，可以使用节点角色显式地配置数据层(内容层、热层、暖层和冷层)，可以使用索引级数据层分配过滤将索引配置为在特定的层中分配。ILM将利用这些层在索引经历其生命周期的各个阶段时自动在节点之间迁移数据。



由数据流抽象的新创建的索引将自动分配给data_hot层，而独立的索引将自动分配给data_content层。具有预先存在的数据角色的节点被认为是所有层的一部分。



### 分类分析的AUC-ROC评价指标

接受者工作特征曲线下面积(AUC ROC)是一个评估指标，从7.3开始就可用于异常值检测，现在可用于分类分析。AUC ROC表示分类过程在不同预测概率阈值下的表现。将特定类的真实阳性率与以不同阈值水平组合的所有其他类的比率进行比较，以创建曲线。



数据帧分析的自定义特征处理器

特性处理器使您能够从文档字段中提取流程特性。您可以在模型培训和模型部署中使用这些特性。自定义特性处理器提供了一种机制来创建可以在搜索和摄取时间使用的特性，而且它们不会占用索引中的空间。这个过程将特性生成与结果模型更紧密地结合在一起。其结果是简化了模型管理，因为特性和模型都可以很容易地遵循相同的生命周期。



### 搜索时间点（PITs）
在7.10中，我们引入了时间点(PITs)，这是一种保存搜索索引状态的轻量级方法。PITs通过使UIs更具反应性来改善最终用户体验。



默认情况下，搜索请求在返回响应之前等待完整的结果。例如，检索顶级命中和聚合的搜索仅在顶级命中和聚合计算完成后才返回响应。然而，聚合通常比top hits更慢，计算成本更高。您可以发送两个单独的请求，而不是发送一个合并的请求:一个用于top hits，另一个用于聚合。使用单独的搜索请求，UI可以在搜索结果可用时立即显示top hits，并在较慢的聚合请求完成后显示聚合数据。您可以使用PIT来确保两个搜索请求运行在相同的数据和索引状态上。



要在搜索中使用PIT，必须首先使用新的开放PIT API显式创建 PIT。keep_alive如果没有后续请求延长其持续时间，则会自动对PIT进行垃圾回收。


```
POST /my-index-000001/_pit?keep_alive=1m
```
API返回您可以在搜索请求中使用的PIT ID。您还可以使用搜索请求的keep_alive参数来配置延长PIT寿命的时间 。

```
POST /_search
{
    "size": 100,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    },
    "pit": {
	    "id":  "46ToAwMDaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQNpZHkFdXVpZDIrBm5vZGVfMwAAAAAAAAAAKgFjA2lkeQV1dWlkMioGbm9kZV8yAAAAAAAAAAAMAWICBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
	    "keep_alive": "1m"
    }
}
```
PIT的keep_alive周期结束时会自动关闭。您也可以使用close PIT API手动关闭不再需要的 PIT。关闭PIT会释放维护PIT索引状态所需的资源。
```
DELETE /_pit
{
    "id" : "46ToAwMDaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQNpZHkFdXVpZDIrBm5vZGVfMwAAAAAAAAAAKgFjA2lkeQV1dWlkMioGbm9kZV8yAAAAAAAAAAAMAWIBBXV1aWQyAAA="
}
```
有关在搜索中使用PIT的更多信息，请参阅通过进行 分页搜索结果 search_after或PIT API文档。

### 协调节点上的请求级断路器编辑
现在，您可以使用协调节点来处理用于在请求断路器中执行部分和最终减少聚合的内存。搜索协调器添加了用于保存和减少请求断路器中的分片聚合结果的内存。在进行任何部分或最终减少之前，将估计减少聚合所需的内存，如果超出此断路器允许的最大内存，则会引发CircuitBreakingException。

估计此大小约为需要减少的序列化聚合大小的1.5倍。对于某些聚合，此估计可能是完全不正确的，但在精简完成后，将使用实际大小对其进行校正。如果缩减成功，我们将更新断路器以删除源聚合的大小，并将估算值替换为新缩减结果的序列化大小。

### EQL：区分大小写和:运算符编辑
在7.10中，默认情况下，我们使大多数EQL运算符和函数区分大小写。我们还添加了:，一个新的不区分大小写的equal运算符。专为安全用例而设计，您可以使用:操作员在Windows事件日志和其他包含字母大小写混合的事件数据中搜索字符串。

```
GET /my-index-000001/_eql/search
{
  "query": """
    process where process.executable : "c:\\\\windows\\\\system32\\\\cmd.exe"
  """
}
``` 
有关更多信息，请参见[EQL语法文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/eql-syntax.html) 。

### REST API对系统索引的访问已被弃用编辑
我们不建议使用REST API访问系统索引。多数尝试访问系统索引的REST API请求都将返回以下弃用警告：
```
this request accesses system indices: [.system_index_name], but in a future
major version, direct access to system indices will be prevented by default
```

以下REST API端点将访问系统索引作为其实现的一部分，并且不会返回弃用警告：

- GET _cluster/health
- GET {index}/_recovery
- GET _cluster/allocation/explain
- GET _cluster/state
- POST _cluster/reroute
- GET {index}/_stats
- GET {index}/_segments
- GET {index}/_shard_stores
- GET _cat/\[indices,aliases,health,recovery,shards,segments\]

我们还添加了一个新的元数据标志来跟踪索引。升级期间，Elasticsearch会自动将此标志添加到任何现有系统索引中。

### 系统索引的新线程池编辑
我们为系统索引添加了两个新的线程池：system_read和 system_write。这些线程池可确保对Elastic Stack至关重要的系统索引（例如安全性或Kibana所使用的系统索引）在集群承受沉重的查询或索引负载时保持响应能力。

system_read是一个fixed线程池，用于管理针对系统索引的读取操作的资源。类似地，system_write是一个 fixed线程池，用于管理针对系统索引的写操作的资源。两者的最大线程数等于5 或等于可用处理器的一半，以较小者为准。

