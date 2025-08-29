---
seo:
  title: 事件流
  description: 事件流是事件处理应用程序的通信机制。您可以使用事件流连接事件处理应用程序。事件流通常被命名并包含已知格式的事件。
---

# 事件流

[事件处理应用程序](../event-processing/event-processing-application.md)需要通信，理想情况下通信基于[事件](../event/event.md)。应用程序需要用于此通信的标准机制。

## 问题

[事件处理器](../event-processing/event-processor.md)和应用程序如何使用事件流相互通信？

## 解决方案
![event-stream](../img/event-stream.svg)

使用事件流连接事件处理应用程序。[事件源](../event-source/event-source.md)向事件流产生事件，事件处理器和[事件接收器](../event-sink/event-sink.md)消费它们。事件流被命名，允许在特定事件流上进行通信。注意事件流如何解耦源和接收器应用程序，它们通过事件间接和异步地相互通信。此外，事件数据格式通常被验证，以管理应用程序之间的通信。

一般来说，事件流将世界上发生的事情的历史记录为事件序列（想想：事实序列）。流的示例将是销售分类账或国际象棋比赛中的移动序列。这个历史是有序的事件序列或链，所以我们知道哪个事件在另一个事件之前发生，并且可以推断因果关系（例如，"白方将e2兵移动到e4；然后黑方将e7兵移动到e5"）。因此，流代表过去和现在：当我们从今天到明天——或从一毫秒到下一毫秒——新事件不断被附加到历史中。

从概念上讲，流提供_不可变_数据。它只支持插入（附加）新事件，现有事件不能被更改。流是持久的、耐用的和容错的。与传统消息队列不同，存储在流中的事件可以被事件接收器和事件处理应用程序根据需要多次读取，并且在消费后不会被删除。相反，保留策略控制事件如何被保留。流中的事件可以是_键控的_，我们可以为一个键有许多事件。对于所有客户的支付流，客户ID可能是键（参见相关模式，如[分区并行性](../event-stream/partitioned-parallelism.md)）。

## 实现

在[Apache Kafka®](/learn-kafka/apache-kafka/events/)中，事件流被称为_主题_。Kafka允许您定义策略来规定如何保留事件，使用[时间或大小限制](../event-storage/limited-retention-event-stream.md)或[永久保留事件](../event-storage/infinite-retention-event-stream.md)。Kafka消费者（事件接收器和事件处理应用程序）能够决定在事件流中的何处开始读取。他们可以选择从最旧或最新的事件开始读取，或使用事件的时间戳或位置（称为_偏移_）寻找到主题中的特定位置。

流技术支持基于Kafka的事件流作为核心抽象。例如：

* [Kafka Streams DSL API](https://kafka.apache.org/30/documentation/streams/developer-guide/dsl-api.html)为事件流提供抽象，特别是用于无界数据流的[`KStream`](https://docs.confluent.io/platform/current/streams/javadocs/javadoc/org/apache/kafka/streams/kstream/KStream.html)接口和用于键控记录变更日志数据的[`KTable`](https://docs.confluent.io/platform/current/streams/javadocs/javadoc/org/apache/kafka/streams/kstream/KTable.html)接口（例如，按产品ID键控的产品更新`KTable`）。
* Apache Flink® [`Table`](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/table/api/Table.html)接口是Flink（Java）Table API用于事件流的核心抽象。PyFlink的Table API也围绕[`Table`](https://pyflink.readthedocs.io/en/main/getting_started/quickstart/table_api.html#Table-Creation)对象构建。
* [Flink SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)使用熟悉的标准SQL语法支持事件流。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息通道](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageChannel.html)。
* 演示如何构建操作事件流的基本流处理应用程序的教程：使用[Kafka Streams](https://developer.confluent.io/confluent-tutorials/creating-first-apache-kafka-streams-application/kstreams/)，使用[Flink SQL](https://developer.confluent.io/confluent-tutorials/filtering/flinksql/)，使用[Flink Table API](https://developer.confluent.io/courses/flink-table-api-java/exercise-connecting-to-confluent-cloud/)。
