---
seo:
  title: 事件流平台
  description: 事件流平台，如Apache Kafka®，允许企业围绕事件流设计流程和应用程序。
---

# 事件流平台

公司很少建立在单一数据存储和与之交互的单一应用程序上。通常，公司可能有数百或数千个应用程序、数据库、数据仓库或其他数据存储。公司的数据分布在这些资源中，它们之间的互连极其复杂。在大型企业中，多条业务线可能使情况更加复杂。现代软件架构，如微服务和SaaS应用程序，也增加了复杂性，因为工程师的任务是将整个基础设施紧密地编织在一起。

此外，公司无法在不实时响应其业务中的[事件](../event/event.md)的情况下生存。客户和业务合作伙伴期望即时反应和丰富的交互式应用程序。今天，数据在运动中，工程团队需要建模应用程序以将业务需求作为事件流处理，而不是作为静态数据，闲置在传统数据存储中。

## 问题

我们可以使用什么架构来将业务中的一切都建模为事件流，为构建现代应用程序创建一个现代、容错和可扩展的平台？

## 解决方案
![event streaming platform](../img/event-streaming-platform.svg)

我们可以围绕[事件流](../event-stream/event-stream.md)设计业务流程和应用程序。从销售、订单、交易和客户体验到传感器读数和数据库更新，一切都建模为[事件](../event/event.md)。事件被写入事件流平台一次，允许业务中的分布式功能实时响应。事件流平台外部的系统使用[事件源](../event-source/event-source.md)和[事件接收器](../event-sink/event-sink.md)进行集成。业务逻辑在[事件处理应用程序](../event-processing/event-processing-application.md)中构建，这些应用程序由从事件流读取事件并向事件流写入事件的[事件处理器](../event-processing/event-processor.md)组成。

## 实现

Apache Kafka®是最流行的事件流平台，旨在满足现代分布式架构的业务需求。您可以使用Kafka以水平可扩展、容错且易于使用的方式读取、写入、处理、查询和响应[事件流](../event-stream/event-stream.md)。Kafka建立在[事件流模式](../index.md)中描述的许多模式之上。

### 基础

Kafka中的数据作为事件交换，事件表示已发生事情的真相。事件的示例包括订单、支付、活动和测量。在Kafka中，事件有时也被称为_记录_或_消息_，它们包含描述事件的数据和元数据。

事件被写入、存储在[事件流](../event-stream/event-stream.md)中并从其中读取。在Kafka中，这些流被称为_主题_。主题有名称，通常包含特定用例的"相关"记录，如客户支付。主题在[事件存储](../event-storage/event-store.md)中被建模为持久、分布式、仅追加的日志。有关Kafka主题的更多详细信息，请参阅[Apache Kafka 101课程](/learn-kafka/apache-kafka/events/)。

向主题写入事件的应用程序称为[生产者](https://docs.confluent.io/platform/current/clients/producer.html)。生产者有多种形式，代表[事件源](../event-source/event-source.md)模式。读取事件由代表[事件接收器](../event-sink/event-sink.md)的[消费者](https://docs.confluent.io/platform/current/clients/consumer.html)执行。消费者通常以分布式、协调的方式运行以提高规模和容错性。[事件处理应用程序](../event-processing/event-processing-application.md)既作为事件源又作为事件接收器。

如上所述产生和消费事件的应用程序被称为"客户端"。这些客户端应用程序可以用各种编程语言编写，包括[Java](/get-started/java)、[Go](/get-started/go)、[C/C++](/get-started/c)、[C# (.NET)](/get-started/dotnet)、[Python](/get-started/python)、[Node.JS](/get-started/nodejs)、[等等](/kafka-languages-and-tools)。

### 流处理

[事件处理应用程序](../event-processing/event-processing-application.md)可以使用各种技术在Kafka之上构建。

#### Apache Flink®

[Flink](https://nightlies.apache.org/flink/flink-docs-stable/)是一个分布式流处理框架，内置支持Kafka主题作为源和接收器。它包括声明式Java和Python API以及支持熟悉的标准SQL语法的[Flink SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)。

#### Kafka Streams

Kafka客户端库[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)允许您使用Java和Scala等语言在JVM上构建弹性应用程序和微服务。应用程序可以在多个实例中以分布式方式运行，以获得更好的可扩展性和容错性。

### 数据集成

[Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)框架允许您可扩展且可靠地将Kafka外部的云服务和数据系统集成到事件流平台中。这些系统的数据通过Kafka_连接器_作为[事件流](../event-stream/event-stream.md)持续导入和/或导出而处于运动中。[Confluent Hub](https://www.confluent.io/hub/)上有数百个即用型Kafka连接器。将现有数据系统引入Kafka通常是采用事件流平台之旅的第一步。

[源连接器](../event-source/event-source-connector.md)从传统数据库、云对象存储服务或Salesforce等SaaS产品等源将数据拉入Kafka主题。可以通过[数据库直写](../event-source/database-write-through.md)和[数据库旁路写入](../event-source/database-write-aside.md)等模式进行高级集成。

[接收器连接器](../event-sink/event-sink-connector.md)是[源连接器](../event-source/event-source.md)的补充模式。虽然源连接器持续将数据带入事件流平台，但接收器持续将数据从Kafka流传递到外部云服务和系统。常见的目标系统包括云数据仓库服务、基于函数的无服务器计算服务、关系数据库、Elasticsearch和云对象存储服务。

## 注意事项

事件流平台是由各种组件组成的分布式计算系统。因为构建和操作这样的平台需要大量的工程专业知识和资源，许多组织选择完全托管的Kafka服务，如[Confluent Cloud](https://www.confluent.io/confluent-cloud/)，而不是自我管理平台，以便他们可以专注于创造业务价值。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息总线](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageBus.html)。
* [Confluent Cloud](https://www.confluent.io/confluent-cloud/)是Apache Kafka®的云原生服务。
* [Apache Kafka 101](/learn-kafka/apache-kafka/)课程提供了关于Kafka是什么以及它如何工作的入门。
