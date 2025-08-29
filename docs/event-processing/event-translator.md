---
seo:
  title: 事件翻译器
  description: 事件翻译器允许使用不同数据格式的系统通过事件进行通信。
---

# 事件翻译器

[事件流平台](../event-stream/event-streaming-platform.md)将随时间连接各种系统，在所有系统中使用通用数据格式可能不可行。

## 问题

使用不同数据格式的系统如何通过[事件](../event/event.md)相互通信？

## 解决方案
![event-translator](../img/event-translator.svg)

事件翻译器将数据格式转换为下游[事件处理器](../event-processing/event-processor.md)熟悉的标准格式。这可以采取字段操作的形式——例如，将一个事件模式映射到另一个事件模式。另一种常见形式是不同的序列化类型——例如，将Apache Avro&trade;转换为JSON或将Protocol Buffers (Protobuf)转换为Avro。

## 实现

使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，我们可以使用SQL语句创建[事件流](../event-stream/event-stream.md)：

```sql
CREATE STREAM translated_stream AS
   SELECT
      fieldX AS fieldC,
      field.Y AS fieldA,
      field.Z AS fieldB
   FROM untranslated_stream;
```

## 注意事项

- 在某些情况下，如果数据丢失，翻译将是单向的。例如，将XML转换为JSON通常会丢失信息，这意味着无法重新创建原始形式。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[事件翻译器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageTranslator.html)。
