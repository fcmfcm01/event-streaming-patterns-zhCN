---
seo:
  title: 模式兼容性
  description: 模式兼容性确保事件可以演化其模式，以便下游应用程序仍然可以处理新旧版本
---

# 模式兼容性

模式就像[数据契约](../event/data-contract.md)一样，它们设定了保证应用程序可以处理其接收数据的条款。
应用程序和数据模式的自然行为是它们随时间演化，因此制定关于如何允许它们演化以及新旧版本之间兼容性规则的政策很重要。

## 问题

我们如何确保模式可以演化而不破坏现有的[事件接收器](../event-sink/event-sink.md)（读者）和[事件源](../event-source/event-source.md)（写入器），包括[事件处理应用程序](../event-processing/event-processing-application.md)？

## 解决方案
![schema-compatibility](../img/schema-compatibility.svg)

有两种类型的兼容性需要考虑：向后兼容性和向前兼容性。

向后兼容性确保较新的_读者_可以更新其模式并仍然消费由较旧写入器编写的事件。
向后兼容变更的类型包括：

* 删除字段：旧写入器可以继续包含此字段，新读者忽略它
* 添加具有默认值的可选字段：旧写入器不写入此字段，新读者使用默认值

向前兼容性确保较新的_写入器_可以产生具有更新模式的事件，这些事件仍然可以被较旧的读者读取。
向前兼容变更的类型包括：

* 添加字段：新写入器包含此字段，旧读者忽略它
* 删除具有默认值的可选字段：新写入器不写入此字段，旧读者使用默认值

## 实现

使用Avro作为[序列化格式](../event/event-serializer.md)，如果原始模式是：

```
{"namespace": "io.confluent.examples.client",
 "type": "record",
 "name": "Event",
 "fields": [
     {"name": "field1", "type": "boolean", "default": true},
     {"name": "field2", "type": "string"}
 ]
}
```

兼容变更的示例将是：

1. _删除具有默认值的字段_：注意`field1`被删除

```
{"namespace": "io.confluent.examples.client",
 "type": "record",
 "name": "Event",
 "fields": [
     {"name": "field2", "type": "string"}
 ]
}
```

2. _添加具有默认值的字段_：注意`field3`被添加，默认值为0。

```
{"namespace": "io.confluent.examples.client",
 "type": "record",
 "name": "Event",
 "fields": [
     {"name": "field1", "type": "boolean", "default": true},
     {"name": "field2", "type": "string"},
     {"name": "field3", "type": "int", "default": 0}
 ]
}
```

## 注意事项

我们可以使用具有内置兼容性检查的[完全托管的模式注册表](https://docs.confluent.io/cloud/current/get-started/schema-registry.html)服务，以便我们可以集中管理模式并检查新模式版本与先前版本的兼容性。

```
curl -X POST --data @filename.avsc https://<schema-registry>/<subject>/versions
```

一旦我们更新了模式并断言了所需的兼容性级别，我们必须仔细考虑使用它们的应用程序的升级顺序。
在某些情况下，我们应该首先升级写入器应用程序（事件源，即消费者），在其他情况下，我们应该首先升级读者应用程序（事件接收器和事件处理器，即生产者）。
有关更多详细信息，请参阅[模式兼容性类型](https://docs.confluent.io/platform/current/schema-registry/avro.html#compatibility-types)。

## 参考资料

* [事件序列化器](../event/event-serializer.md)：编码事件以便可以将它们写入磁盘、通过网络传输，并通常为未来的读者保留
* [读取时模式](../event/schema-on-read.md)：使事件的读者能够确定将哪个模式应用于正在处理的事件
* [模式演化和兼容性](https://docs.confluent.io/platform/current/schema-registry/avro.html)：向后、向前、完全
* [使用模式](https://docs.confluent.io/cloud/current/client-apps/schemas-manage.html)：创建、编辑、比较版本
* [Maven插件](https://docs.confluent.io/platform/current/schema-registry/develop/maven-plugin.html#schema-registry-test-compatibility)在开发周期中测试模式兼容性

