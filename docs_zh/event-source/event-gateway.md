---
seo:
  title: 事件网关
  description: 一个好的事件网关为事件存储的不同用户提供最广泛的访问。
---

# 事件网关

采用[事件](../event/event.md)优先架构的关键好处之一是促进协作。我们的目标是确保团队A可以产生数据，团队B可以处理它，团队C可以报告它，只有一个东西将团队联系在一起——数据本身。团队不应该就共享库、同步发布计划或通用工具达成一致。数据成为唯一真正的接口。

但在现实中，每个团队仍然必须与[事件存储][event_store]本身通信。我们如何最大化访问？我们如何确保每个团队都可以使用事件存储，而不坚持他们从支持语言的短列表中选择？我们如何容纳坚持使用Idris<sup>+</sup>的团队？

<i><sup>+</sup>或Haskell，或Rust，或Erlang，或我们没有想到的任何其他语言...</i>

## 问题

[事件流平台](../event-stream/event-streaming-platform.md)如何为最广泛的用户提供访问？

## 解决方案

![event-gateway](../img/event-gateway.svg)

通过标准化、良好支持的接口提供事件网关，为最广泛的用户提供访问。

## 实现

Confluent提供了一套广泛的[REST API][rest_apis]，允许任何语言或CLI使用HTTP(S)访问事件存储。此外，它还提供支持来生产和消费Apache Kafka®数据，格式为JSON、Protobuf、Avro甚至原始base64编码字节。

作为一个简单的示例，我们可以使用[curl][curl]将JSON编码的事件发布到名为`sales`的主题：

```sh
curl -X POST \
  -H "Content-Type: application/vnd.kafka.json.v2+json" \
  --data '{"records":[{"key":"alice","value":{"tickets":5}},{"key":"bob","value":{"tickets":10}}]}' \
  http://localhost:8082/topics/sales
```

```json
{
  "offsets": [
    {
      "partition": 0,
      "offset": 0,
      "error_code": null,
      "error": null
    },
    {
      "partition": 0,
      "offset": 1,
      "error_code": null,
      "error": null
    }
  ],
  "key_schema_id": null,
  "value_schema_id": null
}
```

## 注意事项

在完美的世界中，每个事件流平台（和每个关系数据库）都会对每种语言提供一流的支持。实际上，某些语言会比其他语言更好地被容纳，但我们仍然可以通过基于标准的接口确保每种语言都能访问每个重要功能。

## 参考资料

* [Confluent REST API][rest_apis]文档

[event_store]: ../event-storage/event-store.md
[rest_apis]: https://docs.confluent.io/platform/current/kafka-rest/index.html
[curl]: https://curl.se/
