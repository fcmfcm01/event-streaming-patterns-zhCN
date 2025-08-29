---
seo:
  title: 有限保留事件流
  description: 有限保留事件流允许从事件流中删除过时或其他不需要的事件。
---

# 有限保留事件流

许多用例允许从[事件流](../event-stream/event-stream.md)中删除[事件](../event/event.md)，以节省空间并防止读取过时数据。

## 问题

我们如何根据标准（如事件年龄或事件流大小）从事件流中删除事件？

## 解决方案
![limited-retention-event-stream](../img/limited-retention-event-stream.svg)

有限保留的解决方案将取决于[事件流平台](../event-stream/event-streaming-platform.md)。大多数平台将允许使用API或在平台本身内配置的自动化过程删除事件。由于事件流在[事件存储](../event-storage/event-store.md)中被建模为不可变事件日志，事件将从事件流的开头开始被删除（即，最旧的事件首先被删除）。从流中读取的[事件处理应用程序](../event-processing/event-processing-application.md)将不会收到已删除的事件。

## 实现

Apache Kafka®默认实现有限保留事件流。使用Kafka，事件流被建模为[主题](https://docs.confluent.io/platform/current/kafka/introduction.html#main-concepts-and-terminology)。Kafka提供两种类型的保留策略，可以按主题配置或作为新主题的默认配置。

### 基于时间的保留

使用基于时间的保留，当事件时间戳指示事件比配置的日志保留时间更旧时，事件将从主题中删除。在Kafka上，这通过`log.retention.hours`设置配置，可以设置为默认值以应用于所有主题或按主题设置。此外，Kafka尊重`log.retention.minutes`和`log.retention.ms`设置来定义更短的保留期。

以下示例将主题的保留期设置为一年：

```bash
log.retention.hours=8760
```

有关基于时间的数据保留的更多指导和配置，请参阅[Kafka代理配置文档](https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html)了解默认设置，或[修改主题](https://docs.confluent.io/platform/current/kafka/post-deployment.html#modifying-topics)部分了解修改现有主题。

### 基于大小的保留

使用基于大小的保留，一旦主题的总大小违反配置的最大大小，事件将开始从主题中删除。Kafka支持`log.retention.bytes`配置。例如，要将主题的最大大小配置为100GB，您可以按如下方式设置配置：

```bash
log.retention.bytes=107374127424
```

有关基于大小的数据保留的更多指导和配置，请参阅[Kafka代理配置文档](https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html)了解默认设置，或[修改主题](https://docs.confluent.io/platform/current/kafka/post-deployment.html#modifying-topics)部分了解修改现有主题。

### 事件如何从事件流中删除

对于配置保留的任一方法，Kafka在事件违反配置的保留设置时_不会立即_逐个删除事件。要了解它们如何被删除，我们首先需要解释Kafka主题进一步分解为_主题分区_（请参阅[分区并行性](../event-stream/partitioned-parallelism.md)）。分区本身进一步分为称为_段_的文件。段表示特定分区中事件的序列，一旦违反保留策略，这些段就会被删除。我们可以通过`log.retention.check.interval.ms`和段配置（如`log.segment.bytes`）等附加设置进一步微调删除算法。

## 注意事项

* 在配置具有有限保留的事件流时，我们应该考虑故障场景。我们的应用程序应该有足够的时间在事件被删除之前读取和处理事件，这意味着配置的保留必须覆盖我们的应用程序可能经历的潜在中断的时间框架。例如，如果运营团队对非关键Kafka用例有2个工作日（周一到周五）的SLA，那么保留应该设置为至少4天，以覆盖在周末之前或期间发生的事件。
* 类似地，如果我们想允许应用程序多次重新处理相同的事件（通常称为_重新处理历史数据_的能力）用于A/B测试或机器学习等用例，那么我们必须相应地增加保留设置。这就是为什么在实践中通常配置具有长保留期（数月或数年）的Kafka主题的原因。

## 参考资料

* 此模式类似于Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息过期](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageExpiration.html)
* [Apache Kafka 101：介绍](/learn-kafka/apache-kafka/events/)提供了"什么是Kafka，它是如何工作的？"的入门
* 相关模式是[无限保留事件流](infinite-retention-event-stream.md)模式，它详细说明了无限期存储事件的事件流。
