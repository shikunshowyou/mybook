# 可伸缩性和弹性:集群、节点和碎片

Elasticsearch的构建目的是随时可用，并根据您的需求进行扩展。它通过自然的分布来做到这一点。您可以向集群中添加服务器(节点)以增加容量，Elasticsearch会自动将数据和查询负载分布到所有可用的节点上。Elasticsearch不需要修改应用程序，它知道如何平衡多节点集群，以提供伸缩性和高可用性。节点越多越好。



这是如何运作的呢?实际上，Elasticsearch索引实际上只是一个或多个物理碎片的逻辑分组，其中每个碎片实际上是一个自包含的索引。通过将文档分布在一个索引中，并将这些分片分布在多个节点上，Elasticsearch可以确保冗余，既可以防止硬件故障，又可以增加集群中节点的查询容量。随着集群的增长(或收缩)，Elasticsearch自动迁移碎片以重新平衡集群。



有两种类型的碎片:基本碎片和副本碎片。索引中的每个文档都属于一个主碎片。复制碎片是主碎片的副本。副本提供数据的冗余副本，以防止硬件故障，并增加处理读取请求(如搜索或检索文档)的容量。



索引中的主碎片数量在创建索引时是固定的，但是复制碎片的数量可以在任何时候更改，而不会中断索引或查询操作。



### 这取决于…

对于为索引配置的分片大小和主分片数量，有许多性能考虑和权衡。碎片越多，维护这些索引的开销就越大。当Elasticsearch需要重新平衡集群时，碎片的大小越大，移动碎片所花费的时间就越长。



查询大量的小碎片会使每个碎片的处理速度更快，但是查询越多就意味着更多的开销，因此查询较少的大碎片可能会更快。简而言之，这要看情况。



作为起点:



- 目标是将平均碎片大小保持在几GB到几十GB之间。对于基于时间的数据的用例，碎片的大小通常在20GB到40GB之间。

- 避免大量的碎片问题。一个节点可以容纳的碎片数量与可用的堆空间成比例。一般来说，每GB堆空间的碎片数量应该少于20个。

为您的用例确定最佳配置的最佳方法是[使用您自己的数据和查询进行测试](https://www.elastic.co/cn/elasticon/conf/2016/sf/quantitative-cluster-sizing) 。



### 万一发生灾难

出于性能方面的原因，集群中的节点需要在同一个网络上。在不同数据中心的节点上平衡集群中的分片花费的时间太长了。但是高可用性架构要求您避免将所有的鸡蛋放在一个篮子里。如果一个位置发生重大故障，另一个位置的服务器需要能够接管。无缝。答案吗?Cross-cluster复制(CCR)。



CCR提供了一种方法，可以将主集群的索引自动同步到可作为热备份的辅助远程集群。如果主集群发生故障，可以由备集群接管。您还可以使用CCR创建辅助集群，以接近您的用户的地理位置为读请求提供服务。



跨集群复制是主备复制。主集群上的索引是活动领导索引，并处理所有写请求。复制到辅助集群的索引是只读追随者。



### 保健和feeding

与任何企业系统一样，您都需要工具来保护、管理和监视Elasticsearch集群。集成到Elasticsearch中的安全性、监视和管理特性使您能够使用Kibana作为管理集群的控制中心。数据汇总和索引生命周期管理等功能可以帮助您随着时间的推移智能地管理数据