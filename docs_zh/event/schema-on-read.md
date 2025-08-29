---
seo:
  title: 读取时模式
  description: 读取时模式使数据读取者能够确定对处理数据应用哪种模式。
---

# 读取时模式
读取时模式将[事件](../event/event.md)模式的验证留给读取者。这种模式有几个用例，它们都为[事件处理器](../event-processing/event-processor.md)和[事件接收器](../event-sink/event-sink.md)提供了很大的灵活性：

1. 可能存在同一模式类型的不同版本，读取者想要选择将哪个版本应用于给定事件。

2. 不同类型事件之间的排序可能很重要，而所有不同的事件类型都被放入单个流中。例如，考虑一个银行用例，客户首先开立账户，然后获得批准，然后进行存款，等等。将这些异构[事件](../event/event.md)类型放入同一流中，允许[事件流平台](../event-stream/event-streaming-platform.md)维护事件排序，并允许任何消费的[事件处理器](../event-processing/event-processor.md)和[事件接收器](../event-sink/event-sink.md)根据需要[反序列化](../event/event-deserializer.md)事件。

3. 可能有非结构化数据写入[事件流](../event-stream/event-stream.md)，然后读取者可以应用它想要的任何模式。

## 问题
[事件处理器](../event-processing/event-processor.md)如何在对[事件流平台](../event-stream/event-streaming-platform.md)读取数据时对数据应用模式？

## 解决方案
![schema-on-read](../img/schema-on-read.svg)

读取时模式方法使每个读取者能够决定如何读取数据，以及将哪个模式的哪个版本应用于它读取的每个[事件](../event/event.md)。
为了使模式管理更容易，设计可以使用集中式存储库来存储不同模式的多个版本，然后客户端应用程序可以在运行时选择将哪个模式应用于事件。

## 实现
使用[Confluent Schema Registry](https://docs.confluent.io/cloud/current/cp-component/schema-reg-cloud-config.html)，所有模式都在集中式存储库中管理。
除了存储模式信息外，Schema Registry还可以配置为检查并强制执行模式更改与先前版本的兼容性。

例如，如果企业从具有两个字段的事件模式定义开始，但企业需求后来演变为需要可选的第三个字段，那么模式会随着企业需求而演进。
Schema Registry确保新模式与旧模式兼容。
在这种特定情况下，为了向后兼容，第三个字段`status`可以用默认值定义，当反序列化用旧模式编码的数据时，将用于缺失字段。
这确保给定流中的所有事件都遵循[模式兼容性](../event-stream/schema-compatibility.md)规则，并且应用程序可以继续处理这些事件。

```
{
 "namespace": "example.avro",
 "type": "record",
 "name": "user",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "address",  "type": "string"},
     {"name": "status", "type": "string", "default": "waiting"}
 ]
}
```

作为另一个示例，如果用例需要将不同的事件类型写入单个流，使用Apache Kafka®，您可以将"主题命名策略"设置为针对记录类型而不是Kafka主题注册模式。
Schema Registry然后将允许在每种事件类型的范围内而不是主题范围内执行[模式演进](../event-stream/schema-evolution.md)和[模式兼容性](../event-stream/schema-compatibility.md)验证。

消费者应用程序可以读取分配给数据类型的模式版本，如果任何给定流中有不同的数据类型，应用程序可以在处理时将每个事件转换为适当的类型，然后遵循适当的代码路径：

```java
if (Account.equals(record.getClass()) {
  ...
} else if (Approval.equals(record.getClass())) {
  ...
} else if (Transaction.equals(record.getClass())) {
  ...
} else {
  ...
}
```

## 注意事项
模式的主题命名策略可以通过两种方式之一设置为记录类型（而不是Kafka主题）。
限制较少的是`RecordNameStrategy`，它将命名空间设置为记录，无论事件写入到哪个主题。
限制较多的是`TopicRecordNameStrategy`，它将命名空间设置为记录和事件写入的主题。

## 参考资料
* [您应该将几种事件类型放在同一个Kafka主题中吗？](https://www.confluent.io/blog/put-several-event-types-kafka-topic/)
* [Confluent Schema Registry文档](https://docs.confluent.io/cloud/current/cp-component/schema-reg-cloud-config.html)
