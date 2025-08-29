---
seo:
  title: 事件时间处理
  description: 事件时间处理允许事件流应用程序使用事件最初发生的时间戳来处理事件。
---

# 事件时间处理

一致的时间语义在流处理中特别重要。[事件处理器](../event-processing/event-processor.md)中的许多操作都依赖于时间，如连接、在时间窗口上计算的聚合（例如，五分钟平均值）以及处理乱序和"延迟"数据。在许多系统中，开发人员可以为事件选择不同的时间变体：

1. 事件时间，捕获[事件源](../event-source/event-source.md)最初创建事件的时间。
2. 摄取时间，捕获[事件流平台](../event-processing/event-processing-application.md)中事件流接收事件的时间。
3. 挂钟时间或处理时间，下游[事件处理器](../event-processing/event-processor.md)处理事件的时间（可能在事件时间后几毫秒、几小时、几个月或更长时间）。

根据用例，开发人员需要选择一种变体而不是其他变体。

## 问题

我们如何实现基于事件时间的事件处理（即基于每个事件原始时间线的处理）？

## 解决方案

![event-time-processing](../img/event-time-processing.svg)

对于事件时间处理，[事件源](../event-source/event-source.md)必须在每个事件中包含时间戳（例如，在数据字段或头元数据中），表示事件源创建事件的时间。然后，在消费端，[事件处理应用程序](../event-processing/event-processing-application.md)需要从事件中提取此时间戳。这允许应用程序基于其原始时间线处理事件。

## 实现

### Apache Flink®

在Flink中，基于[事件时间处理](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/)的流应用程序需要指定事件时间水印。水印用于信号和跟踪时间的流逝，可用于实现处理或丢弃延迟事件的宽限期。

例如，以下Flink SQL表定义定义了[水印策略](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/create/#watermark)，使得发出的水印是最大观察事件时间戳减去30秒。这种延迟水印策略有效地允许事件比迄今为止看到的事件晚30秒。

```sql
CREATE TABLE orders (
    order_id INT,
    item_id INT,
    ts TIMESTAMP(3),
    WATERMARK FOR ts AS ts - INTERVAL '30' SECOND
);
```

### Kafka Streams

Apache Kafka的Kafka Streams客户端库提供了`TimestampExtractor`接口来从事件中提取时间戳。默认实现从Kafka消息中检索时间戳（参见上面的讨论），如消息生产者设置的那样。通常，这种设置会产生事件时间处理，这正是我们想要的。

但对于那些需要从事件负载中获取时间戳的情况，我们可以创建自己的`TimestampExtractor`实现：

```java
class OrderTimestampExtractor implements TimestampExtractor {
@Override
public long extract(ConsumerRecord<Object, Object> record, long partitionTime) {
    ElectronicOrder order = (ElectronicOrder)record.value();
    return order.getTime();
}
```

一般来说，这种自定义时间戳分配功能使得集成来自不使用Kafka Streams的其他应用程序的数据变得容易。

此外，Kafka有事件时间与处理时间（挂钟时间）与摄取时间的概念。像Kafka Streams这样的客户端使得在我们的应用程序中选择我们想要使用的时间变体成为可能。

## 注意事项

在决定使用哪种时间语义时，我们需要考虑问题域。在大多数情况下，事件时间处理是推荐的选项。例如，当重新处理历史事件流（如用于A/B测试、训练机器学习模型）时，只有事件时间处理产生正确的结果。如果我们使用处理时间（挂钟时间）来处理过去四周的事件，那么[事件处理器](../event-processing/event-processor.md)会错误地认为这四周的数据刚刚在几分钟内创建，这完全破坏了原始时间线和数据的时间分布，从而导致不正确的处理结果。

事件时间和摄取时间之间的差异通常不太明显，但摄取时间仍然遭受事件在现实世界中实际发生的时间（事件时间）与事件在[事件流平台](../event-processing/event-processing-application.md)中被接收和存储的时间（摄取时间）之间的相同概念差异。如果由于某种原因，事件被捕获和交付到事件流平台之间存在显著延迟，那么事件时间是更好的选项。

不使用事件时间的一个原因是我们不能信任[事件源](../event-source/event-source.md)为我们提供可靠的数据，包括事件的可靠嵌入时间戳。在这种情况下，如果修复根本原因（不可靠的事件源）不可行，摄取时间可能是首选选项。

## 参考资料

* 另请参阅[挂钟时间处理](../stream-processing/wallclock-time.md)模式，它提供了关于使用当前时间（挂钟时间）作为事件时间的更多详细信息。
