---
seo:
  title: 事件反序列化器
  description: 事件反序列化器将已编码存储的事件转换为可用的、特定于语言的数据结构。
---

# 事件反序列化器

数据有很长的生命周期，通常比最初收集和存储它的程序寿命更长。数据来自各种系统和编程语言。我们越容易访问这些数据海洋，我们就能进行越丰富的分析。

在在线购物业务中，订单处理系统记录的数据和用户行为数据可能对网站设计部门非常宝贵，_前提是他们能够实际访问它_。能够从[事件存储](../event-storage/event-store.md)读取数据是至关重要的，无论最初是哪个过程和哪个部门将其放在那里。

在很大程度上，数据的可访问性在写入时由我们对[事件序列化器](event-serializer.md)的选择决定。然而，在我们读取数据之前，故事当然还没有完成。

## 问题

如何从事件流平台中的表示重建原始事件？

## 解决方案

![event deserializer](../img/event-deserializer.svg)

使用与模式注册表集成良好的[事件流平台](../event-stream/event-streaming-platform.md)。这使得鼓励（或要求）写入者记录事件的数据描述以供以后使用变得容易。同时拥有事件数据及其模式使得反序列化变得容易。

虽然某些数据格式是合理的[可发现的](https://en.wikipedia.org/wiki/Discoverability)，但在实践中，拥有数据在写入时如何编码的精确、永久记录变得非常宝贵。如果数据格式随时间演变，并且[事件流](../event-stream/event-stream.md)可能包含语义等效数据的多种编码，这一点尤其重要。

## 实现

Confluent的[Schema Registry](https://docs.confluent.io/cloud/current/cp-component/schema-reg-cloud-config.html)在Apache Kafka®本身中存储数据模式的版本化历史。客户端库然后可以使用此元数据无缝重建原始事件数据，而我们可以使用注册表API手动检查模式，或为其他语言构建库。

例如，在[事件序列化器](event-serializer.md)模式中，我们编写了一个`fx_trades`事件表。如果我们想回忆这些事件的结构，我们可以请求Flink SQL表定义：

```sh
DESCRIBE fx_trades;
```

```text
+---------------+----------------+-------+-----+--------+-----------+
|          name |           type |  null | key | extras | watermark |
+---------------+----------------+-------+-----+--------+-----------+
|      trade_id |            INT | FALSE |     |        |           |
| from_currency |     VARCHAR(3) |  TRUE |     |        |           |
|   to_currency |     VARCHAR(3) |  TRUE |     |        |           |
|         price | DECIMAL(10, 5) |  TRUE |     |        |           |
+---------------+----------------+-------+-----+--------+-----------+
```

或者我们可以直接查询Schema Registry以机器可读格式查看结构：

```sh
curl http://localhost:8081/subjects/fx_trades-value/versions/latest | jq .```
```

```json
{
  "subject": "fx_trades-value",
  "version": 1,
  "id": 1,
  "schema": "{\"type\":\"record\",\"name\":\"record\",\"namespace\":\"org.apache.flink.avro.generated\",\"fields\":[{\"name\":\"from_currency\",\"type\":[\"null\",\"string\"],\"default\":null},{\"name\":\"to_currency\",\"type\":[\"null\",\"string\"],\"default\":null},{\"name\":\"price\",\"type\":[\"null\",{\"type\":\"bytes\",\"logicalType\":\"decimal\",\"precision\":10,\"scale\":5}],\"default\":null}]}"
}
```

解包该`schema`字段揭示了[Avro][avro]规范：

```sh
curl http://localhost:8081/subjects/fx_trades-value/versions/latest | jq -rc .schema | jq .
```

```json
{
  "type": "record",
  "name": "record",
  "namespace": "org.apache.flink.avro.generated",
  "fields": [
    {
      "name": "from_currency",
      "type": [
        "null",
        "string"
      ],
      "default": null
    },
    {
      "name": "to_currency",
      "type": [
        "null",
        "string"
      ],
      "default": null
    },
    {
      "name": "price",
      "type": [
        "null",
        {
          "type": "bytes",
          "logicalType": "decimal",
          "precision": 10,
          "scale": 5
        }
      ],
      "default": null
    }
  ]
}
```

Avro库可以使用此模式无缝反序列化事件。任何支持Schema Registry的客户端库都可以自动化此查找，允许我们完全忘记编码并专注于数据。

## 注意事项

除了Avro，Schema Registry还支持Protobuf和JSON Schema。有关这些格式的讨论，请参阅[事件序列化器](event-serializer.md)。

虽然序列化格式的选择很重要，但它不必一成不变。例如，使用Kafka Streams在支持的格式之间进行转换是直接的。对于更复杂的场景，我们有几种管理模式迁移的策略：

* [模式兼容性](../event-stream/schema-evolution.md)讨论了Avro设计为透明处理的"安全"模式更改类型。
* [事件转换器](../event-processing/event-translator.md)可以在不同编码之间转换，以帮助不同系统的消费。
* [模式演进](../event-stream/schema-evolution.md)讨论了分割和连接流，以简化只能处理事件模式某些版本的消费者的服务。
* [事件标准化器](event-standardizer.md)可以将不同的数据编码重新格式化为单一的统一格式。
* 我们总是可以选择使用[读取时模式](schema-on-read.md)策略直接在代码中处理编码问题。

## 参考资料

* 事件反序列化器（用于读取）的对应物是[事件序列化器](event-serializer.md)（用于写入）。
* 序列化器和反序列化器与[数据契约](data-contract.md)密切相关，其中我们希望遵循特定的序列化格式，_并且_将各个事件限制在该格式内的某个模式。
* 另请参阅：[事件映射器](../event-processing/event-mapper.md)。

[Avro]: https://avro.apache.org/docs/current/
