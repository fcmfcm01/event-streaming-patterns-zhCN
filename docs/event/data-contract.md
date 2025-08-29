---
seo:
  title: 数据契约
  description: 数据契约确保当事件处理应用程序发送事件时，接收应用程序知道如何处理它。
---

# 数据契约

[事件处理应用程序](../event-processing/event-processing-application.md)可以向另一个事件处理应用程序发送[事件](../event/event.md)。通信的应用程序理解如何处理这些共享事件是至关重要的。

## 问题
应用程序如何知道如何处理由另一个应用程序发送的[事件](../event/event.md)？

## 解决方案
![data-contract](../img/data-contract.svg)

使用数据契约或模式，不同的[事件处理应用程序](../event-processing/event-processing-application.md)可以共享[事件](../event/event.md)并理解如何处理它们，而发送应用程序和接收应用程序不需要了解彼此的详细信息。数据契约模式允许这些不同的应用程序协作，同时保持松散耦合，从而免受它们可能实施的任何内部更改的影响。通过实施数据契约或模式，您可以提供与关系数据库管理系统（RDBMS）相同的记录一致性保证，后者默认集成模式。

## 实现

通过使用模式来建模事件对象，Apache Kafka®客户端（如Kafka生产者、Kafka Streams应用程序或[Apache Flink®](https://nightlies.apache.org/flink/flink-docs-stable/)应用程序）可以理解如何处理来自使用相同模式的不同应用程序的事件。
例如，我们可以使用Apache Avro来描述模式：
```json
{
  "type":"record",
  "namespace": "io.confluent.developer.avro",
  "name":"Purchase",
  "fields": [
    {"name": "item", "type":"string"},
    {"name": "amount", "type": "double"},
    {"name": "customer_id", "type": "string"}
  ]
}
```

此外，使用中央存储库，如Confluent [Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)，使Kafka客户端能够轻松利用模式。

## 注意事项

与其为数据契约或模式实施自定义支持，不如考虑使用行业接受的模式支持框架，如下所示：

* [Apache Avro](https://avro.apache.org/docs/current/spec.html)
* [Protocol Buffers](https://developers.google.com/protocol-buffers)（Protobuf）
* [JSON Schema](https://json-schema.org/)。

## 参考资料
* [是的，弗吉尼亚，你真的需要一个模式注册表](https://www.confluent.io/blog/schema-registry-kafka-stream-processing-yes-virginia-you-really-need-one/)
