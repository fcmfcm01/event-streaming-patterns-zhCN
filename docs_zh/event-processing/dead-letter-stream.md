---
seo:
  title: 死信流
  description: 死信流是处理事件处理应用程序中错误的模式。错误类别包括：无效数据格式、技术故障、缺失或损坏的值，或其他意外场景。
---

# 死信流

[事件处理应用程序](event-processing-application.md)在处理无限事件流时可能遇到无效数据。错误可能包括无效数据格式、无意义的、缺失或损坏的值、技术故障，或其他意外场景。

## 问题

当消息无法读取时，事件处理应用程序如何在不终止或卡住的情况下处理处理失败？

## 解决方案
![dead-letter-stream](../img/dead-letter-stream.svg)

当事件处理应用程序由于不可恢复的原因无法处理事件时，有问题的事件被发布到"死信流"。此流存储事件，允许对其进行记录、稍后重新处理或以其他方式处理。可以在"死信事件"中提供额外的上下文信息，以方便稍后的故障解决，例如关于其处理失败原因的详细信息。

## 实现

Java基本Kafka消费者
```java
while (keepConsuming) {
  try {
    final ConsumerRecords<K, V> records = consumer.poll(Duration.ofSeconds(1));
    try {
      eventProcessor.process(records);
    } catch (Exception ex) {
      deadEventReporter.report(/*Error Details*/);
    }
  }
  catch (SerializationException se) {
    deadEventReporter.report(/*Error Details*/);
  }
}
```

Python基本Kafka消费者
```python
while True:
    try:
        event = consumer.poll(1000)

    except SerializerError as e:
        deadEventReporter.report(e)
        break

    if msg.error():
        deadEventReporter.report(msg.error())
        continue

    if msg is None:
        continue

    eventProcessor.process(msg)
```

## 注意事项

现实世界的应用程序应该如何处理死信流中的事件？自动重新处理事件通常会导致重新排序，因此如果流包含表示同一底层实体变化状态的事件（如订单被预订、处理或发货），则可能对下游系统造成损坏。手动重新处理可能有用，但在许多现实世界的实现中，通常更多地被视为错误日志。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[死信通道](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html)
* Confluent的[模式注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)提供数据治理功能，包括"写入时模式"强制执行，这可以帮助下游消费者免受意外事件格式的影响。
* Kafka Streams提供了注册自定义Serde来处理损坏记录的能力，允许创建可以发布到死信流的死信数据。有关详细信息，请参阅此Confluent [Kafka Streams FAQ](https://docs.confluent.io/platform/current/streams/faq.html#streams-faq-failure-handling-deserialization-errors-serde)。
