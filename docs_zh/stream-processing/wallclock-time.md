---
seo:
  title: 挂钟时间处理
  description: 处理来自事件源的事件，考虑处理时间戳、挂钟时间和处理时间的不同方式。
---

# 挂钟时间处理

一致的时间语义在流处理中特别重要。[事件处理器](../event-processing/event-processor.md)中的许多操作都依赖于时间，如连接、在时间窗口上计算的聚合（例如，五分钟平均值）以及处理乱序和"延迟"数据。在许多系统中，开发人员可以为[事件](../event/event.md)选择不同的时间变体：

1. 事件时间，捕获事件源最初创建事件的时间。
2. 摄取时间，捕获[事件流平台](../event-stream/event-streaming-platform.md)中事件流接收事件的时间。
3. 挂钟时间或处理时间，下游事件处理器处理事件的时间（可能在事件时间后几毫秒、几小时、几个月或更长时间）。

根据用例，开发人员需要选择一种变体而不是其他变体。

## 问题

如何独立于事件源最初创建事件时的时间戳来处理来自[事件源](../event-source/event-source.md)的事件？

## 解决方案
![wallclock-time](../img/wallclock-time.svg)

根据用例，[事件处理器](../event-processing/event-processor.md)可以使用[事件源](../event-source/event-source.md)最初创建事件的时间、[事件流平台](../event-stream/event-streaming-platform.md)中事件流接收事件的时间，或从[事件](../event/event.md)本身提供的一个或多个数据字段（即从事件负载）派生的时间。

## 实现

作为示例，[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)使用系统`PROCTIME()`函数将挂钟处理时间作为计算列公开。

```sql
CREATE TABLE device_readings (
    device_id INT,
    temperature DOUBLE,
    wallclock_time AS PROCTIME()
);

SELECT device_id,
    COUNT(*) AS reading_count,
    window_start,
    window_end
FROM TABLE(TUMBLE(TABLE device_readings, DESCRIPTOR(wallclock_time), INTERVAL '5' MINUTES))
GROUP BY device_id, window_start, window_end;
```

## 参考资料

* [事件时间处理](../stream-processing/event-time-processing.md)模式提供了关于流处理中时间语义的基本信息。
