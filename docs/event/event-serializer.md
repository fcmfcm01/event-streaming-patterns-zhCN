---
seo:
  title: 事件序列化器
  description: 事件序列化器编码事件，使其可以写入磁盘、通过网络传输，并通常为未来的读者保存。
---

# 事件序列化器

数据有很长的生命周期，通常比最初收集和存储它的程序寿命更长。数据需要广泛的受众——我们的数据越容易访问，我们组织中的更多部门就能找到它的用途。

在一个成功的系统中，销售部门在一年中收集的数据可能在几年后对营销部门非常宝贵，_前提是他们能够实际访问它_。

为了最大效用和长寿性，数据应该以不向未来的读者和写入者隐藏的方式写入。数据比今天的技术选择更重要。

这如何影响基于事件的系统？这种架构是否有任何特殊考虑，或者编程语言的序列化工具就足够了？

## 问题

我如何将事件转换为事件流平台和使用它的应用程序理解的格式？

## 解决方案

![event serializer](../img/event-serializer.svg)

使用与语言无关的序列化格式。理想的格式应该是自文档化的、空间高效的，并设计为支持某种程度的向后和向前兼容性。我们推荐[Avro][avro]。（参见"注意事项"。）

一个可选的、推荐的步骤是在模式注册表中注册序列化详细信息。注册表为[事件反序列化器](event-deserializer.md)和[模式验证器](../event-source/schema-validator.md)提供可靠的、机器可读的参考点，使事件消费变得非常简单。

## 实现

例如，我们可以使用Avro为外汇交易定义结构：

```json
{"namespace": "io.confluent.developer",
 "type": "record",
 "name": "FxTrade",
 "fields": [
     {"name": "trade_id", "type": "long"},
     {"name": "from_currency", "type": "string"},
     {"name": "to_currency", "type": "string"},
     {"name": "price", "type": "bytes", "logicalType": "decimal", "precision": 10, "scale": 5}
 ]
}
```

...然后使用我们语言的Avro库来处理序列化：

```java
  FxTrade fxTrade = new FxTrade( ... );

  ProducerRecord<long, FxTrade> producerRecord =
    new ProducerRecord<>("fx_trade", fxTrade.getTradeId(), fxTrade);

  producer.send(producerRecord);
```

或者，使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，我们可以定义[事件流](../event-stream/event-stream.md)的方式强制执行该格式并使用Confluent的[Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)记录Avro定义：

```sql
CREATE TABLE fx_trades (
    trade_id INT NOT NULL,
    from_currency VARCHAR(3),
    to_currency VARCHAR(3),
    price DECIMAL(10,5)
) WITH (
    'connector' = 'kafka',
    'topic' = 'fx_trades',
    'properties.bootstrap.servers' = 'broker:9092',
    'scan.startup.mode' = 'earliest-offset',
    'key.format' = 'raw',
    'key.fields' = 'trade_id',
    'value.format' = 'avro-confluent',
    'value.avro-confluent.url' = 'http://schema-registry:8081',
    'value.fields-include' = 'EXCEPT_KEY'
);
```

通过此设置，数据的序列化和反序列化都由Flink在后台自动执行。

## 注意事项

[事件流平台](../event-stream/event-streaming-platform.md)通常与序列化无关，接受从人类可读文本到原始字节的任何序列化数据。然而，通过限制自己使用更广泛接受的结构化数据格式，我们可以为与其他项目和编程语言的更容易协作打开大门。

找到"通用"序列化格式不是一个新问题，也不是事件流独有的问题。因此，我们有多种与技术无关的解决方案随时可用。简要介绍几个：

* [JSON](https://www.json.org/)。可以说是计算历史上最成功的序列化格式。JSON是一种基于文本的格式，易于读取、写入和[发现](https://en.wikipedia.org/wiki/Discoverability)<sup>1</sup>，这从世界各地以最少协作生产和消费JSON数据的语言和项目数量可以看出。
* [Protocol buffers](https://developers.google.com/protocol-buffers)。由Google支持并由各种语言支持，Protobuf是一种二进制格式，牺牲了JSON的可发现性，以获得更紧凑的表示，使用更少的磁盘空间和网络带宽。Protobuf也是一种强类型格式，允许从写入者强制执行特定的数据模式，并向读取者描述该数据的结构。
* [Avro][avro]。一种类似于Protocol Buffers的二进制格式，Avro的设计专注于支持模式的演进，允许数据格式随时间变化，同时最小化对未来读者和写入者的影响。

虽然序列化格式的选择很重要，但它不必一成不变。使用Kafka Streams在支持的格式之间进行转换是直接的。对于更复杂的场景，我们有几种管理模式迁移的策略：

* [模式兼容性](../event-stream/schema-compatibility.md)讨论了Avro设计为透明处理的"安全"模式更改类型。
* [事件转换器](../event-processing/event-translator.md)可以在不同编码之间转换，以帮助不同系统的消费。
* [模式演进](../event-stream/schema-evolution.md)讨论了分割和连接流，以简化只能处理事件模式某些版本的消费者的服务。
* [事件标准化器](event-standardizer.md)可以将不同的数据编码重新格式化为单一的统一格式。
* 作为后备方案，我们可以使用[读取时模式](schema-on-read.md)策略将问题推给消费者的代码。

## 参考资料

* 事件序列化器（用于写入）的对应物是[事件反序列化器](event-deserializer.md)（用于读取）。
* 序列化器和反序列化器与[数据契约](data-contract.md)密切相关，其中我们希望遵循特定的序列化格式，_并且_将各个事件限制在该格式内的某个模式。
* 另请参阅：[事件映射器](../event-processing/event-mapper.md)。

## 脚注

_<sup>1</sup> 年长的程序员会讲述80年代银行使用的较难发现的序列化格式的故事，其中破译消息的含义意味着翻阅厚厚的、环装的打印输出数据规范，该规范通过交叉引用"编码子格式22"来解释"字段78"的含义。_

[Avro]: https://avro.apache.org/docs/current/
