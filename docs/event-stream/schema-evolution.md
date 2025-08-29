---
seo:
  title: 模式演化
  description: 模式演化为事件添加新信息或重新构建事件。
---

# 模式演化

模式演化是数据管理的重要方面。
类似于API如何演化并需要与依赖较旧或较新API版本的所有应用程序兼容，模式也会演化，同样需要与依赖较旧或较新模式版本的所有应用程序兼容。

## 问题

我如何重新构建事件或向事件添加新信息，理想情况下以确保[模式兼容性](schema-compatibility.md)的方式？

## 解决方案
![schema-evolution](../img/schema-evolution-1.svg)

演化模式的一种方法是"就地"演化模式（如上图所示）。在这种方法中，流可以包含使用新旧模式版本的事件。然后模式兼容性检查确保[事件处理应用程序](../event-processing/event-processing-application.md)和[事件接收器](../event-sink/event-sink.md)可以读取两种格式的模式。

![schema-evolution](../img/schema-evolution-2.svg)

另一种方法是执行_双重模式升级_，也称为创建_版本化流_（如上图所示）。当您需要向流的模式引入破坏性变更且新模式与先前模式不兼容时，这种方法特别有用。在这里，[事件源](../event-source/event-source.md)写入两个流：

1. 一个使用先前模式版本的流，如`payments-v1`。
2. 另一个使用新模式版本的流，如`payments-v2`。

然后事件处理应用程序和事件接收器各自从使用与其兼容的模式版本的流中消费。
在将所有消费者升级到新模式版本后，您可以停用使用旧模式版本的流。

## 实现

对于"就地"模式演化，[事件流](../event-stream/event-stream.md)具有同一模式的多个版本，不同版本[相互兼容](schema-compatibility.md)。
例如，如果删除了一个字段，那么为处理没有此字段的事件而开发的消费者将能够处理使用旧模式编写并包含该字段的事件；消费者将简单地忽略该字段。
在这种情况下，如果原始模式如下：

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

然后我们可以修改模式以省略`field2`，如下所示，事件流将混合两种模式类型。
如果新消费者处理使用旧模式编写的事件，这些消费者将忽略`field2`。

```
{"namespace": "io.confluent.examples.client",
 "type": "record",
 "name": "Event",
 "fields": [
     {"name": "field1", "type": "boolean", "default": true}
 ]
}
```

_双重模式升级_涉及破坏性变更，因此处理流程需要更明确：客户端必须只写入和读取与其可以处理的模式对应的事件流。
例如，可以处理v1模式的应用程序将从一个流读取：

```sql
CREATE STREAM payments-v1 (transaction_id BIGINT, username VARCHAR)
    WITH (kafka_topic='payments-v1');
```

可以处理v2模式的应用程序将从另一个流读取：

```sql
CREATE STREAM payments-v2 (transaction_id BIGINT, store_id VARCHAR)
    WITH (kafka_topic='payments-v2');
```

## 注意事项

遵循[模式兼容性](../event-stream/schema-compatibility.md)规则来确定哪些模式变更是兼容的，哪些是破坏性变更。

## 参考资料

* 另请参阅[事件序列化器](../event/event-serializer.md)模式，其中我们编码事件以便可以将它们写入磁盘、通过网络传输，并通常为未来的读者保留
* 另请参阅[读取时模式](../event/schema-on-read.md)模式，它使事件的读者能够确定将哪个模式应用于读者正在处理的事件。
* [模式演化和兼容性](https://docs.confluent.io/platform/current/schema-registry/avro.html)涵盖向后兼容性、向前兼容性和完全兼容性。
* [使用模式](https://docs.confluent.io/cloud/current/client-apps/schemas-manage.html)涵盖如何创建模式、编辑它们和比较版本。
* [模式注册表Maven插件](https://docs.confluent.io/platform/current/schema-registry/develop/maven-plugin.html#schema-registry-test-compatibility)允许您在开发周期中测试模式兼容性。
