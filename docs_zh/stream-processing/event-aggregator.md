---
seo:
  title: 事件聚合器
  description: 对多个相关事件执行聚合以产生新事件。
---

# 事件聚合器

将多个事件组合成单个包含事件——例如，计算总数、平均值等——是事件流和流分析中的常见任务。

## 问题

如何聚合多个相关事件以产生新事件？

## 解决方案

我们使用[事件分组器](../stream-processing/event-grouper.md)后跟事件聚合器。分组器根据需要为后续聚合步骤准备输入事件，例如，通过基于计算聚合的数据字段（如客户ID）对事件进行分组和/或通过将事件分组到时间窗口（如5分钟窗口）中。然后聚合器为每组事件计算所需的聚合，例如，通过计算每个5分钟窗口的平均值或总和。

## 实现
![event-aggregator](../img/event-aggregator.svg)

例如，我们可以使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)和Apache Kafka®来执行聚合。

假设我们有一个基于现有Kafka主题的名为`orders`的Flink SQL表：

```sql
CREATE TABLE orders (
    order_id INT,
    item_id INT,
    total_units INT,
    ts TIMESTAMP(3),
    WATERMARK FOR ts AS ts
);
```

然后我们将创建一个包含来自该流的聚合事件的派生表。在这种情况下，我们创建一个名为`item_stats`的表，表示1小时窗口内每个项目的订单统计：

```sql
CREATE TABLE item_stats AS
  SELECT item_id,
      COUNT(*) AS total_orders,
      AVG(total_units) AS avg_units,
      window_start,
      window_end
  FROM TABLE(TUMBLE(TABLE orders, DESCRIPTOR(ts), INTERVAL '1' HOURS))
  GROUP BY item_id, window_start, window_end;
```

每当新事件到达`orders`表时，此表将持续更新。

## 注意事项

* 在事件流中，一个关键技术挑战是——除了少数例外——通常不可能判断输入数据在给定时间点是否"完整"。因此，像Apache Kafka的Kafka Streams客户端库这样的流处理技术采用诸如_松弛时间_<sup>1</sup>和_宽限期_等技术（例如，参见Kafka Streams [`ofSizeAndGrace`](https://kafka.apache.org/38/javadoc/org/apache/kafka/streams/kstream/TimeWindows.html#ofSizeAndGrace(java.time.Duration,java.time.Duration))方法用于在窗口操作中指定宽限期）。Apache Flink®水印和关联的水印策略定义了截止点，之后[事件处理器](../event-processing/event-processor.md)将从其处理中丢弃任何延迟到达的输入事件，例如，参见延迟水印策略Flink SQL示例[这里](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/concepts/time_attributes/)。有关更多信息，请参阅[抑制事件聚合器](../stream-processing/suppressed-event-aggregator.md)模式。

## 参考资料

* 相关模式：[抑制事件聚合器](../stream-processing/suppressed-event-aggregator.md)
* 事件聚合的更详细示例可以在以下教程中看到：[如何使用Kafka Streams计算事件流](https://developer.confluent.io/confluent-tutorials/aggregating-count/kstreams/)和[如何使用Flink SQL计算事件流](https://developer.confluent.io/confluent-tutorials/aggregating-count/flinksql/)。

## 脚注

<sup>1</sup>松弛时间：[超越分析：流处理系统的演变（SIGMOD 2020）](https://dl.acm.org/doi/abs/10.1145/3318464.3383131)，[Aurora：数据流管理的新模型和架构（VLDB Journal 2003）](https://dl.acm.org/doi/10.1007/s00778-003-0095-z)
