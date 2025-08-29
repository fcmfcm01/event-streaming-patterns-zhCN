---
seo:
  title: 幂等写入器
  description: 幂等写入器向事件流平台精确一次地产生事件。
---

# 幂等写入器

写入器产生被写入[事件流](../event-stream/event-stream.md)的[事件](../event/event.md)，在稳定条件下，每个事件只被记录一次。
但是，在操作故障或短暂网络中断的情况下，[事件源](../event-source/event-source.md)可能需要重试写入。这可能导致相同事件的多个副本最终出现在事件流中，因为第一次写入可能实际上已经成功，即使生产者没有收到[事件流平台](../event-stream/event-streaming-platform.md)的确认。这种类型的重复是实践中的常见故障场景，也是分布式系统的危险之一。

## 问题

[事件流平台](../event-stream/event-streaming-platform.md)如何确保事件源不会多次写入相同的事件？

## 解决方案
![idempotent-writer](../img/idempotent-writer.svg)

一般来说，这可以通过对幂等客户端的原生支持来解决。
这意味着写入器可能尝试多次产生事件，但事件流平台检测并丢弃对相同事件的重复写入尝试。

## 实现

要使Apache Kafka®生产者幂等，请使用以下设置配置您的生产者：

```
enable.idempotence=true
```

Kafka生产者为其发送到Kafka集群的每批事件标记序列号。集群中的代理使用此序列号来强制执行来自此特定生产者的事件的去重。每批的序列号被持久化，因此即使[领导者代理](https://www.confluent.io/blog/apache-kafka-intro-how-kafka-works/#replication)失败，新的领导者代理也会知道给定的批次是否是重复的。

要在使用Kafka源和接收器的Apache Flink®应用程序中启用精确一次处理，请将交付保证配置为精确一次，如果应用程序使用DataStream Kafka连接器，则通过[`DeliveryGuarantee.EXACTLY_ONCE`](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/kafka/#fault-tolerance) `KafkaSink`配置选项，或者如果它使用Table API连接器之一，则将[`sink.delivery-guarantee`](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/table/kafka/#consistency-guarantees)配置选项设置为`exactly-once`。[Confluent Cloud for Apache Flink](https://docs.confluent.io/cloud/current/flink/overview.html)提供内置的精确一次语义。在Flink应用程序的下游，请确保将任何Kafka消费者配置为[`isolation.level`](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html#isolation-level)为`read_committed`，因为Flink利用嵌入式生产者中的Kafka事务来实现精确一次处理。

要在Kafka Streams或ksqlDB中启用[精确一次处理保证](https://docs.confluent.io/platform/current/installation/configuration/streams-configs.html#processing-guarantee)，请使用以下设置配置应用程序，这包括在嵌入式生产者中启用幂等性：

```
processing.guarantee=exactly_once_v2
```

## 注意事项

为Kafka生产者启用幂等性不仅确保重复事件被从主题中隔离，还确保它们按顺序写入。这是因为代理只有在批次的序列号恰好比最后提交批次的序列号大一时才接受一批事件；否则，会导致顺序错误。

精确一次语义（EOS）允许[事件流应用程序](../event-processing/event-processing-application.md)处理数据而不丢失或重复。这确保计算结果始终一致和准确，即使对于有状态计算，如连接、[聚合](../stream-processing/event-aggregator.md)和[窗口化](../stream-processing/event-grouper.md)。任何需要EOS保证的解决方案必须在管道的所有阶段启用EOS，而不仅仅是在写入器上。因此，幂等写入器通常与[幂等读取器](../event-processing/idempotent-reader.md)和事务处理结合使用。

## 参考资料

* 关于Apache Kafka中精确一次语义的[博客文章](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
* 关于Apache Flink与Apache Kafka作为源和接收器的端到端精确一次处理的[博客文章](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)
