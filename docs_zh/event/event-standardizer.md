---
seo:
  title: 事件标准化器
  description: 事件标准化器将多种格式的事件转换为下游事件处理器理解的通用格式。
---

# 事件标准化器
在大多数企业中，各种传统和[事件处理应用程序](../event-processing/event-processing-application.md)需要通过[事件流](../event-stream/event-stream.md)交换[事件](../event/event.md)。下游事件处理应用程序将需要标准化的数据格式以正确处理这些事件。然而，拥有这些事件的许多来源的现实往往导致缺乏此类标准，或对同一标准的不同解释。

## 问题
我如何处理语义等效但以不同格式到达的事件？

## 解决方案
![event-standardizer](../img/event-standardizer.svg)
将所有输入事件流源到事件标准化器中，该标准化器将事件传递给专门的[事件转换器](../event-processing/event-translator.md)，后者又将事件转换为下游[事件处理器](../event-processing/event-processor.md)理解的通用格式。

## 实现
作为示例，我们可以使用Apache Kafka®的[Kafka Streams客户端库](https://docs.confluent.io/platform/current/streams/index.html)构建一个事件处理应用程序，该应用程序从多个输入事件流读取，然后将值映射到新类型。具体来说，我们可以使用`mapValues`函数将每种事件类型转换为输出[事件流](../event-stream/event-stream.md)上期望的标准类型。

```
SpecificAvroSerde<SpecificRecord> inputValueSerde = constructSerde();

builder
  .stream(List.of("inputStreamA", "inputStreamB", "inputStreamC"),
    Consumed.with(Serdes.String(), inputValueSerde))
  .mapValues((eventKey, eventValue) -> {
    if (eventValue.getClass() == TypeA.class)
      return typeATranslator.normalize(eventValue);
    else if (eventValue.getClass() == TypeB.class)
      return typeBTranslator.normalize(eventValue);
    else if (eventValue.getClass() == TypeC.class)
      return typeCTranslator.normalize(eventValue);
    else {
      // exception or dead letter stream
    }
  })
  .to("my-standardized-output-stream", Produced.with(Serdes.String(), outputSerdeType));
```

## 注意事项
* 在可能的情况下，不同的数据格式应该"在源头"标准化。这种数据治理通常称为"写入时模式"，可以使用[模式验证器](../event-source/schema-validator.md)模式实施。在将事件写入事件流之前强制执行模式验证，允许消费应用程序将其数据格式验证逻辑委托给模式验证层。
* 在标准化器设计中应考虑错误处理。错误类别可能包括序列化失败、意外或缺失值以及未知类型（如上述示例中所示）。[死信流](../event-processing/dead-letter-stream.md)是事件处理应用程序中处理异常事件的常用模式之一。

## 参考资料
* 另请参阅[流合并器](../stream-processing/event-stream-merger.md)模式，用于统一相关流而_不_改变其格式。
* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[标准化器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Normalizer.html)。
* Kafka Streams [`map`无状态转换](https://docs.confluent.io/platform/current/streams/developer-guide/dsl-api.html#creating-source-streams-from-ak)文档
* [Apache Kafka应用程序的错误处理模式](https://www.confluent.io/blog/error-handling-patterns-in-kafka/)是一篇博客文章，详细介绍了[事件处理应用程序](../event-processing/event-processing-application.md)中错误处理的策略和模式
