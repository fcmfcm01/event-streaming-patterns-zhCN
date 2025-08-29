---
seo:
   title: 事件路由器
   description: 事件路由器用于根据每个事件中包含的数据或元数据值将事件路由到不同的事件流。
---

# 事件路由器

[事件流](../event-stream/event-stream.md)可能包含需要隔离处理的[事件](../event/event.md)子集。例如，库存检查系统可能分布在多个物理系统上，目标系统可能取决于被检查项目的类别。

## 问题

我们如何根据事件的属性将[事件](../event/event.md)隔离到专用的[事件流](../event-stream/event-stream.md)中？

## 解决方案
![event-router](../img/event-router.svg)

## 实现

作为示例，使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，我们可以使用`CREATE TABLE AS SELECT`（CTAS）语法持续将事件路由到不同的事件流：

```sql
CREATE TABLE payments ...;

CREATE TABLE payments_france AS
    SELECT * FROM payments WHERE country = 'france';

CREATE TABLE payments_spain AS
    SELECT * FROM payments WHERE country = 'spain';
```

使用Apache Kafka®客户端库[Kafka Streams](https://kafka.apache.org/documentation/streams/)，使用[`TopicNameExtractor`](https://kafka.apache.org/38/javadoc/org/apache/kafka/streams/processor/TopicNameExtractor.html)将事件路由到不同的事件流（在Kafka中称为"主题"）。`TopicNameExtractor`有一个要实现的方法`extract()`，它接受三个参数：

- 事件键
- 事件值
- [`RecordContext`](https://kafka.apache.org/38/javadoc/org/apache/kafka/streams/processor/RecordContext.html)，它提供对事件的头、分区和其他上下文信息的访问

我们可以使用任何给定的参数来为给定事件生成并返回所需的目标主题名称。Kafka Streams将完成路由。

```java
CountryTopicExtractor implements TopicNameExtractor<String, String> {
   String extract(String key, String value, RecordContext recordContext) {
      switch (value.country) {
        case "france":
          return "france-topic";
        case "spain":
          return "spain-topic";
      }
   }
}

KStream<String, String> myStream = builder.stream(...);
myStream.mapValues(..).to(new CountryTopicExtractor());
```

## 注意事项

* 事件路由器不应修改事件本身，而应仅提供到所需目标的适当路由。
* 如果事件路由器需要向事件附加额外信息或上下文，请考虑使用[事件信封](../event/event-envelope.md)模式。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息路由器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageRouter.html)。
* 有关在运行时动态路由事件的完整示例，请参阅教程[如何在运行时动态选择输出主题](https://developer.confluent.io/confluent-tutorials/dynamic-output-topic/kstreams/)。
