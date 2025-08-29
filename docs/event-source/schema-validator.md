---
seo:
  title: 模式验证器
  description: 模式验证器为发送到事件流的所有事件强制执行定义的模式
---

# 模式验证器

在[事件流平台](../event-stream/event-streaming-platform.md)中，创建和写入[事件](../event/event.md)的[事件源](../event-source/event-source.md)与读取和处理事件的[事件接收器](../event-sink/event-sink.md)和[事件处理应用程序](../event-processing/event-processing-application.md)解耦。确保事件生产者和消费者之间的互操作性要求他们就事件的数据模式达成一致。这是为数据治理建立[数据契约](../event/data-contract.md)的重要方面。

## 问题

如何为发送到[事件流](../event-stream/event-stream.md)的所有[事件](../event/event.md)强制执行定义的模式？

## 解决方案
![schema-validator](../img/schema-validator.svg)

在将[事件](../event/event.md)写入流之前，验证事件是否符合[事件流](../event-stream/event-stream.md)的已定义模式。您可以通过两种方式执行此模式验证：

1. 在服务器端，接收[事件](../event/event.md)的[事件流平台](../event-stream/event-streaming-platform.md)可以验证事件。如果事件模式验证失败并因此违反[数据契约](../event/data-contract.md)，事件流平台可以拒绝该事件。
2. 在客户端，创建[事件](../event/event.md)的[事件源](../event-source/event-source.md)可以验证事件。例如，[事件源连接器](../event-source/event-source-connector.md)可以在将事件摄取到[事件流平台](../event-stream/event-streaming-platform.md)之前验证事件，或者[事件处理应用程序](../event-processing/event-processing-application.md)可以使用支持模式的序列化库（如Confluent的Kafka序列化器和反序列化器）提供的模式验证功能。

## 实现

使用Confluent，模式验证通过每个环境管理的[模式注册表](https://docs.confluent.io/platform/current/schema-registry/index.html)得到完全支持。使用Confluent Cloud控制台在您选择的云提供商中启用模式注册表。可以使用Confluent Cloud控制台或[Confluent CLI](https://docs.confluent.io/confluent-cli/current/overview.html)按主题管理模式。以下命令使用CLI创建模式：

```
confluent schema-registry schema create --subject employees-value --schema employees.avsc --type AVRO
```

## 注意事项

* 模式验证器是"写入时模式"的数据治理实现，在事件发布之前强制执行数据一致性。替代策略是[读取时模式](../event/schema-on-read.md)，其中在写入时不强制执行数据格式，但要求消费事件处理应用程序在读取每个事件时验证数据格式。
* 当您想要在组织内部集中强制执行此模式时，服务器端模式验证是优选的。相比之下，客户端验证假设客户端应用程序及其开发人员的合作，这可能或可能不被接受（例如，在受监管的行业中）。
* 模式验证导致负载增加，因为它影响每个事件的写入路径。客户端验证主要影响客户端应用程序的负载。服务器端模式验证增加了事件流平台的负载，而客户端应用程序受影响较小（这里，主要影响是处理被拒绝的事件；请参阅[死信流](../event-processing/dead-letter-stream.md)）。

## 参考资料

* 有关模式如何随时间演变而继续被验证的信息，请参阅[模式兼容性](../event-stream/schema-compatibility.md)模式。
* 了解更多关于如何[使用Confluent和Kafka管理和验证模式](https://docs.confluent.io/cloud/current/client-apps/schemas-manage.html)。
