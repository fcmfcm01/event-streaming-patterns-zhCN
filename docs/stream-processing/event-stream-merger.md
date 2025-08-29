---
seo:
  title: 事件流合并器
  description: 事件流合并器将来自多个事件流的事件合并到单个事件流中，而不改变基础数据。
---

# 事件流合并器

[事件流应用程序](../event-processing/event-processing-application.md)可能包含多个[事件流](../event-stream/event-stream.md)实例。但在某些情况下，应用程序将单独的事件流合并到单个事件流中可能是有意义的，而不改变单个事件。虽然这可能看起来在逻辑上与连接相关，但这种合并是完全不同的操作。连接通过组合具有相同键的事件来产生结果，产生可能不同类型的新事件。事件流的合并将来自多个事件流的事件组合到单个事件流中，但单个事件保持不变并保持相互独立。

## 问题

应用程序如何合并单独的事件流？

## 解决方案
![event-stream-merger](../img/event-stream-merger.svg)

## 实现

对于Apache Kafka®，Kafka Streams客户端库在其DSL中提供`merge`操作符。此操作符将两个事件流合并到单个事件流中。然后我们可以获取合并的流并对其执行任意数量的操作。

```java
KStream<String, Event> eventStream = builder.stream(...);
KStream<String, Event> eventStreamII = builder.stream(...);
KStream<String, Event> allEventsStream = eventStream.merge(eventStreamII);

allEventsStream.groupByKey()...
```

## 注意事项

* Kafka Streams不保证来自基础事件流的记录的处理顺序。
* 为了合并多个事件流，它们必须使用相同的键和值类型。

## 参考资料

* [如何使用Kafka Streams将多个流合并为一个流](https://developer.confluent.io/confluent-tutorials/merging/kstreams/)
* [如何使用Apache Flink® SQL将多个流合并为一个流](https://developer.confluent.io/confluent-tutorials/merging/flinksql/)
