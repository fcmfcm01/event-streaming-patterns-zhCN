---
seo:
  title: 幂等读取器
  description: 幂等读取器可以消费相同的事件一次或多次，产生相同的效果。
---

# 幂等读取器

在理想情况下，[事件](../event/event.md)只会被写入[事件流](../event-stream/event-stream.md)一次。在正常操作下，流的所有消费者也只会读取和处理每个事件一次。但是，根据[事件源](../event-source/event-source.md)的行为和配置，可能会有故障导致重复事件。当这种情况发生时，我们需要一个处理重复事件的策略。

[幂等](https://en.wikipedia.org/wiki/Idempotence)读取器必须考虑重复事件的两个原因：

1. *操作故障*：在分布式系统中，间歇性网络和系统故障是不可避免的。在机器故障或短暂网络中断的情况下，[事件源](../event-source/event-source.md)可能由于重试而产生相同事件的多次。类似地，[事件接收器](../event-sink/event-sink.md)可能由于间歇性偏移更新故障而多次消费和处理相同的事件。[事件流平台](../event-stream/event-streaming-platform.md)应该通过提供强交付和处理保证（如Apache Kafka®事务中的保证）来自动防范这些操作故障。

2. *错误的应用程序逻辑*：[事件源](../event-source/event-source.md)可能错误地多次产生相同的事件，从[事件流平台](../event-stream/event-streaming-platform.md)的角度来看，这会在[事件流](../event-stream/event-stream.md)中填充多个不同的事件。例如，想象事件源中的一个bug，它写入客户支付事件两次，而不是只写入一次。事件流平台对业务逻辑一无所知，因此无法区分这两个事件，而是将它们视为两个不同的支付事件。

## 问题

应用程序在从事件流读取时如何处理重复事件？

## 解决方案
![idempotent-reader](../img/idempotent-reader.svg)

这可以通过精确一次语义（EOS）来解决，包括对事务的原生支持和对幂等客户端的支持。
EOS允许[事件流应用程序](../event-processing/event-processing-application.md)处理数据而不丢失或重复，确保计算结果始终准确。

[事件源](../event-source/event-source.md)的[幂等写入](idempotent-writer.md)是解决此问题的第一步。幂等写入提供了生产者事件的强精确一次交付保证，并消除了操作故障作为写入重复事件的原因。

在读取端，在[事件处理器](../event-processing/event-processor.md)和[事件接收器](../event-sink/event-sink.md)中，可以配置幂等读取器只读取已提交的事务。这防止读取不完整事务中的事件，为读取器提供与操作写入器故障的隔离。请记住，幂等性意味着读取器的业务逻辑必须能够多次处理相同的消费事件，其中多次读取具有与单次读取事件相同的效果。例如，如果读取器管理它已读取事件的计数器（即状态），那么多次读取相同事件应该只增加计数器一次。

由上游源的错误应用程序逻辑引起的重复最好通过修复应用程序的逻辑（即修复根本原因）来解决。在不可能的情况下，例如当事件在我们控制之外生成时，下一个最佳选择是在事件中嵌入跟踪ID。跟踪ID应该是逻辑事件唯一的字段，例如事件键或请求ID。然后消费者可以读取跟踪ID，将其与已处理ID的内部状态存储进行交叉引用，并在必要时丢弃事件。

## 实现

要在Apache Kafka中配置幂等读取器只读取已提交的事务，请在Kafka消费者上设置以下参数：

```
isolation.level="read_committed"
```

如果Apache Flink®应用程序在幂等读取器的上游，请将交付保证配置为精确一次，如果应用程序使用DataStream Kafka连接器，则通过[`DeliveryGuarantee.EXACTLY_ONCE`](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/kafka/#fault-tolerance) `KafkaSink`配置选项，或者如果它使用Table API Kafka连接器之一，则将[`sink.delivery-guarantee`](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/table/kafka/#consistency-guarantees)配置选项设置为`exactly-once`。[Confluent Cloud for Apache Flink](https://docs.confluent.io/cloud/current/flink/overview.html)提供内置的精确一次语义。

如果Kafka Streams或ksqlDB应用程序在幂等读取器的上游，您可以[启用精确一次处理](https://docs.confluent.io/platform/current/streams/developer-guide/config-streams.html#processing-guarantee)。在单个事务内，使用精确一次处理的Kafka Streams应用程序将原子地更新其消费者偏移、其状态存储（包括其变更日志主题）、其重新分区主题和其输出主题。
要启用EOS，请使用以下参数配置您的应用程序：

```
processing.guarantee="exactly_once_v2"
```

要处理错误的应用程序逻辑，首先尝试从代码中消除重复的来源。如果这不是一个选项，您可以根据事件的内容为每个事件分配跟踪ID，使消费者能够自己检测重复。这要求每个消费者应用程序维护一个内部状态存储来跟踪事件的唯一ID，此存储的大小将根据事件数量和消费者必须防范重复的时期而变化。此选项需要额外的磁盘使用量和处理能力来插入和验证事件。

对于业务案例的子集，也可能设计消费者处理逻辑为幂等。例如，在简单的ETL中，数据库是[事件接收器](../event-sink/event-sink.md)，重复事件可以在数据库对事件ID作为主键进行`upsert`期间被去重。幂等业务逻辑还使您的应用程序能够从不是严格无重复的[事件流](../event-stream/event-stream.md)读取，或从在创建时可能没有EOS可用的历史流读取。

## 注意事项

需要EOS保证的解决方案必须在管道的所有阶段启用EOS，而不仅仅是在读取器上。因此，幂等读取器通常与[幂等写入器](../event-processing/idempotent-writer.md)和事务处理结合使用。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[幂等接收器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html)。
* 关于Apache Kafka中精确一次语义的[博客文章](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
* 关于Apache Flink与Apache Kafka作为源和接收器的端到端精确一次处理的[博客文章](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)
