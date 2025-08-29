---
seo:
  title: 无限保留事件流
  description: 无限保留事件流描述无限期保留事件，使事件流平台成为记录系统。
---

# 无限保留事件流

许多用例要求[事件流](../event-stream/event-stream.md)中的[事件](../event/event.md)将被永久存储，以便数据集完整可用。

## 问题

我们如何确保流中的事件被永久保留？

## 解决方案
![infinite-retention-event-stream](../img/infinite-stream-strorage.svg)

无限保留的解决方案取决于特定的[事件流平台](../event-stream/event-streaming-platform.md)。一些平台"开箱即用"支持无限保留，不需要最终用户采取任何行动。如果[事件流平台](../event-stream/event-streaming-platform.md)不支持无限存储，可以通过[事件接收器连接器](../event-sink/event-sink-connector.md)模式部分实现无限保留，该模式将[事件](../event/event.md)卸载到永久外部存储中。

## 实现

使用[Confluent Cloud](https://www.confluent.io/confluent-cloud/)时，无限保留内置于[事件流平台](../event-stream/event-streaming-platform.md)中（_可用性可能基于集群类型和云提供商而受到限制_）。平台用户可以受益于无限存储，而无需对其客户端应用程序或操作进行任何更改。

对于本地[事件流平台](../event-stream/event-streaming-platform.md)，[Confluent Platform](https://www.confluent.io/product/confluent-platform/)通过使用[分层存储](https://docs.confluent.io/platform/current/kafka/tiered-storage.html)扩展Apache Kafka来增加无限保留的能力。分层存储分离计算和存储层，允许操作员根据需要独立扩展其中任何一个。新到达的[事件](../event/event.md)被认为是"热的"，但随着时间推移，它们变得"更冷"并迁移到更具成本效益的外部存储，如AWS S3存储桶。由于云原生对象存储可以有效地扩展到无限大小，Kafka集群可以作为无限[事件流](../event-stream/event-stream.md)的记录系统。

## 注意事项

* 无限保留流通常用于存储将被许多订阅者使用的完整数据集。例如，在无限保留事件流中存储规范客户数据集使其对任何其他系统可用，无论其数据库技术如何。客户数据集可以轻松地整体导入或重新导入。

* [压缩事件流](../event-storage/compacted-event-stream.md)通常用作无限保留事件流的一种形式。但是压缩流不是无限的。相反，它们只保留每个键的最新[事件](../event/event.md)，这意味着它们的内容匹配等效CRUD数据库表中保存的数据集。

## 参考资料

* 博客文章[Confluent中的无限存储](https://www.confluent.io/blog/infinite-kafka-storage-in-confluent-platform/)更详细地描述了分层存储方法。
* [事件接收器连接器](../event-sink/event-sink-connector.md)可用于通过将[事件](../event/event.md)加载到永久外部存储来实现无限保留事件流。
