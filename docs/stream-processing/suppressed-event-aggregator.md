---
seo:
  title: 抑制事件聚合器
  description: 抑制事件聚合器是只提供最终聚合结果而不包括"中间"结果的聚合器
---

# 抑制事件聚合器

[事件流应用程序](../event-processing/event-processing-application.md)可以执行像[事件聚合器](event-aggregator.md)一样的连续聚合操作。但是，通常聚合器会发出"中间"处理结果。这是因为事件流可能是无限的，所以我们通常不知道何时输入被认为是"完整的"。因此，除了少数例外，我们真的没有一个可以获得"最终"结果的点。此外，这种发出策略的好处是在接收到新输入事件和可用更新处理结果之间产生低延迟。

然而，在某些情况下，我们更愿意接收单一的最终结果，而不是多个中间结果。例如，当我们必须将聚合结果输入到本质上与流式方法不兼容的系统中时。在这里，我们需要一个能够"抑制"中间结果的聚合器，以便只产生单一的最终结果。

## 问题

事件聚合器如何提供最终聚合结果，而不是不断更新的"中间"结果？

## 解决方案
![suppressed-event-aggregator](../img/suppressed-event-aggregator.svg)

一般来说，这只对窗口聚合是可能的。这是因为，在这种情况下，聚合器确实知道给定窗口的输入（例如，计算5分钟平均值的5分钟窗口）何时被认为是完整的，因此可以配置为抑制"中间"结果，直到窗口时间过去。我们如何做到这一点？

首先，聚合器的输入事件必须通过[事件分组器](../stream-processing/event-grouper.md)进行窗口化，即事件根据其时间戳被分组到"窗口"中。根据配置的分组，事件可以专门放置到单个窗口中，或者可以放置到多个窗口中。
然后，事件聚合器对每个窗口执行其操作。

只有当窗口被认为已经"过去"（例如，从上午9:00开始到上午9:05结束的5分钟窗口）时，聚合器才会为该窗口输出单一的最终结果。例如，考虑客户支付事件流的聚合，我们希望计算每小时的支付数量。通过使用1小时的窗口大小，一旦相应的1小时窗口关闭，我们就可以发出每小时支付数量的最终计数。

理想情况下，聚合器能够处理乱序或"延迟"事件，这是事件流平台中需要处理的常见情况（例如，在9:03创建的事件只在9:07到达）。在这里，一种常见的技术是让用户为窗口定义所谓的_宽限期_，给延迟事件一些额外的时间到达。在窗口宽限期内到达的事件将被处理，而任何更晚的事件将不被处理（例如，如果宽限期是3分钟，那么9:07到达的9:03事件将包含在9:00-9:05窗口中；任何在9:08或之后到达的事件将被此窗口忽略）。

请注意，使用宽限期会增加处理延迟，因为聚合器必须等待额外的时间段才能知道给定窗口的输入是完整的，因此才能为该窗口输出单一的最终结果。

## 实现

对于Apache Kafka®，[Kafka Streams客户端库](https://docs.confluent.io/platform/current/streams/index.html)在其DSL中提供了`suppress`操作符，我们可以将其应用于窗口聚合。

在以下示例中，我们在订单流上计算每小时聚合，使用5分钟的宽限期等待任何稍微延迟到达的订单。`suppress`操作符确保每个小时窗口只有一个结果事件。

```java
KStream<String, OrderEvent> orderStream = builder.stream(...);

 orderStream.groupByKey()
            .windowedBy(TimeWindows.of(Duration.ofHours(1)).grace(Duration.ofMinutes(5)))
            .aggregate(() -> 0.0 /* 每个窗口`total`的初始值 */,
                       (key, order, total) -> total + order.getPrice(),
                       Materialized.with(Serdes.String(), Serdes.Double()))
            .suppress(untilWindowCloses(unbounded()))
            .toStream()
            .map((windowKey, value) -> KeyValue.pair(windowKey.key(),value))
            .to(outputTopic, Produced.with(Serdes.String(), Serdes.Double()));
```

## 注意事项

* 为了遵守每个窗口只输出单一结果的约定，抑制聚合器通常以某种方式缓冲事件，直到窗口关闭。如果其实现使用内存缓冲区，那么根据每个窗口的事件数量及其负载大小，存在遇到内存不足错误的风险。
