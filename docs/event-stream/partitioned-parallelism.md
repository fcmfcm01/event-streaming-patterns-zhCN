---
seo:
  title: 分区并行性
  description: 分区是并行性的单位，支持大规模并发读取、写入和处理事件。
---

# 分区并行性

如果服务目标要求高吞吐量，能够分发事件存储以及事件生产和消费以进行并行处理是有用的。
分发和并发处理事件使应用程序能够扩展。

## 问题

我如何在[事件流](../event-stream/event-stream.md)和[表](../table/state-table.md)中分配事件，以便它们可以被分布式[事件处理器](../event-processing/event-processor.md)并发处理？

## 解决方案
![partitioned-parallelism](../img/partitioned-parallelism.svg)

使用分区事件流，然后将事件分配给流的不同分区。本质上，分区是用于存储、读取、写入和处理事件的并行性单位。分区通过两种主要方式实现并发性和可扩展性：

* 平台可扩展性：不同的[事件代理](../event-stream/event-broker.md)可以并发存储[事件](../event/event.md)并为[事件处理应用程序](../event-processing/event-processing-application.md)提供服务
* 应用程序可扩展性：不同的[事件处理应用程序](../event-processing/event-processing-application.md)可以并发处理[事件](../event/event.md)

事件分区还影响应用程序语义：将事件放置到给定分区中保证每个分区内事件的_排序_得到保留（但通常不在同一流的不同分区之间）。这种排序保证对许多用例至关重要；事件的排序通常很重要（例如，在处理零售订单时，订单必须在发货前付款）。

## 实现

使用Apache Kafka®，流（称为_主题_）由管理员或流应用程序创建。分区数量在创建主题时指定。例如：

```
confluent kafka topic create myTopic --partitions 30
```

[事件](../event/event.md)根据[事件源](../event-source/event-source.md)的分区算法（如[事件处理应用程序](../event-processing/event-processing-application.md)）放置到特定分区中。
分配给给定分区的所有事件都有强排序保证。

常见的分区方案有：

1. 基于事件键的分区（如客户支付流的客户ID），其中具有相同键的事件存储在同一分区中
2. 轮询分区，为每个分区提供事件的均匀分布
3. 为特定用例定制的自定义分区算法

在基于Kafka的技术中，如[Kafka Streams应用程序](https://docs.confluent.io/platform/current/streams/index.html)或使用其Kafka连接器之一的[Apache Flink®](https://flink.apache.org/)，处理器可以通过并发和分布式方式处理一组分区来扩展。如果事件流的键内容由于查询处理行的方式而改变——例如，在Kafka Streams中执行两个事件流之间的连接——底层键被重新计算，事件被发送到新主题中的新分区以执行计算。（这种内部操作通常称为_分布式数据洗牌_。）

## 注意事项

一般来说，更多的流分区导致更高的吞吐量。为了最大化吞吐量，您需要足够的分区来利用[事件处理器](../event-processing/event-processor.md)的所有分布式实例（例如，Kafka Streams应用程序实例）。
请确保根据[事件源](../event-source/event-source.md)（如Kafka生产者，包括连接器）、[事件处理器](../event-processing/event-processor.md)（如Kafka Streams或Flink应用程序）和[事件接收器](../event-sink/event-sink.md)（如Kafka消费者，包括连接器）的吞吐量仔细选择分区数量。还要确保在环境中进行性能基准测试。
规划数据模式和键分配的设计，以便事件在流分区中尽可能均匀分布。
这将防止某些流分区相对于其他流分区过载。请参阅博客文章[Apache Kafka中的流和表：弹性、容错和其他高级概念](https://www.confluent.io/blog/kafka-streams-tables-part-4-elasticity-fault-tolerance-advanced-concepts/)以了解更多关于分区和处理分区倾斜的信息。

## 参考资料

* 博客文章[如何在Kafka集群中选择主题/分区数量？](https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster)为选择主题的分区数量提供了有用的指导。
* 对于将工作单元从分区细分到事件或事件键的处理并行性方法，请参阅[Confluent Kafka并行消费者](https://github.com/confluentinc/parallel-consumer)。
