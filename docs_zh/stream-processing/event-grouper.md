---
seo:
  title: 事件分组器
  description: 事件分组器是使用聚合函数按事件中的公共属性将事件分组在一起的事件处理器。
---

# 事件分组器

事件分组器是[事件处理器](../event-processing/event-processor.md)的专门形式，它通过公共字段（如客户ID）和/或事件时间戳（通常称为_窗口化_或_基于时间的窗口化_）将事件分组在一起。

## 问题

我们如何将来自同一[事件流](../event-stream/event-stream.md)或[表](../table/state-table.md)的单独但相关的事件分组，以便它们随后可以作为整体处理？

## 解决方案
![event-grouper](../img/event-grouper.svg)

对于_基于时间的分组_（也称为_基于时间的窗口化_），我们使用[事件处理器](../event-processing/event-processor.md)，根据事件时间戳将相关事件分组到窗口中。大多数窗口类型都有预定义的窗口大小，如10分钟或24小时。会话窗口是一个例外，其中每个窗口的大小根据分组事件的时间特征而变化。

对于_基于字段的_分组，我们使用事件处理器，通过一个或多个数据字段对事件进行分组，而不考虑事件时间戳。

两种分组方法是正交的，可以组合。例如，要计算支付流中每个客户的7天平均值，我们首先按客户ID_和_7天窗口对流中的事件进行分组，然后计算每个客户+窗口分组的相应平均值。

## 实现

作为示例，[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)提供了按列对相关事件进行分组并将它们分组到窗口中的能力，其中所有相关事件的时间戳都在定义的时间窗口内。

```sql
SELECT product_name, SUM(price) as total
FROM TABLE(TUMBLE(TABLE purchases, DESCRIPTOR(ts), INTERVAL '1' MINUTES))
GROUP BY product_name, window_start, window_end;
```

## 注意事项

当将事件分组到时间窗口中时，有各种可能的分组类型。

* 跳跃窗口基于时间间隔。它们建模固定大小的、可能重叠的窗口。跳跃窗口由两个属性定义：窗口的持续时间和其前进或"跳跃"间隔。
* 翻滚窗口是跳跃窗口的特例。像跳跃窗口一样，翻滚窗口基于时间间隔。它们建模固定大小、不重叠、无间隙的窗口。翻滚窗口由单个属性定义：窗口的持续时间。
* 会话窗口将事件聚合到会话中，会话表示由指定的非活动间隙或"空闲"分隔的活动期间。时间戳发生在现有会话非活动间隙内的任何记录都会合并到现有会话中。如果记录的时间戳发生在会话间隙之外，则创建新会话。

有关窗口类型的详细信息和图表说明，请参阅[Flink SQL窗口化表值函数](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/queries/window-tvf/)和[Kafka Streams支持的窗口类型](https://docs.confluent.io/platform/current/streams/developer-guide/dsl-api.html#streams-developer-guide-dsl-windowing)。

## 参考资料

* [Apache Flink® SQL中的翻滚窗口](https://developer.confluent.io/confluent-tutorials/tumbling-windows/flinksql/)和[Kafka Streams中的翻滚窗口](https://developer.confluent.io/confluent-tutorials/tumbling-windows/kstreams/)教程提供了在事件窗口上计算聚合计算的端到端示例。
* 相关的完整教程包括[Flink SQL中的会话窗口](https://developer.confluent.io/confluent-tutorials/session-windows/flinksql/)和[Kafka Streams中的会话窗口](https://developer.confluent.io/confluent-tutorials/session-windows/kstreams/)，以及[Flink SQL中的跳跃窗口](https://developer.confluent.io/confluent-tutorials/hopping-windows/flinksql/)。
