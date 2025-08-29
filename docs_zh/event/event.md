---
seo:
  title: 事件
  description: 事件表示事实，被解耦的应用程序、服务和系统用于在事件流平台上交换数据。
---

# 事件
事件表示事实，被解耦的应用程序、服务和系统用于在[事件流平台](../event-stream/event-streaming-platform.md)上交换数据。

## 问题
如何表示已发生事情的事实？

## 解决方案
![event](../img/event.svg)
事件表示已发生事情的不可变事实。事件的例子可能包括订单、支付、活动或测量数据。事件被生产到、存储在[事件流](../event-stream/event-stream.md)中，并从其中被消费。事件通常包含描述事实的一个或多个数据字段，以及表示事件被其[事件源](../event-source/event-source.md)创建时的时间戳。事件还可能包含各种元数据，如其来源（例如，创建事件的应用程序或云服务）和存储级信息（例如，其在事件流中的位置）。

## 实现
在Apache Kafka®中，事件被称为_记录_。记录被建模为具有时间戳和可选元数据（称为头部）的键/值对。记录的_值_通常包含应用程序域对象的表示或某种形式的原始消息值，例如传感器的输出或其他指标读数。记录_键_有几个用途，但关键的是，Kafka使用它来确定数据如何在流中分区，也称为_主题_（有关分区的更多详细信息，请参阅[分区并行性](../event-stream/partitioned-parallelism.md)）。_键_通常最好被视为事件的分类，如特定用户或连接设备的身份。头部是记录元数据的位置，可以帮助描述事件数据本身，它们本身被建模为键和值的_映射_。

记录键、值和头部是不透明的数据类型，这意味着Kafka为了达到其高可扩展性和性能，故意设计不为其定义类型接口：它们被Kafka的服务器端代理作为原始字节数组读取、存储和写入。Kafka_客户端_应用程序（如[Apache Flink®](https://developer.confluent.io/courses/apache-flink/intro/)流处理应用程序，或使用客户端库实现的微服务，如[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)或[Kafka Go客户端](https://docs.confluent.io/kafka-clients/go/current/overview.html)）负责对记录键、值和头部中的数据进行序列化和反序列化。

当使用Java客户端库时，事件使用`ProducerRecord`类型创建，并使用`KafkaProducer`发送到Kafka。在此示例中，我们将键和值类型设置为字符串，并添加了一个头部：

```java
ProducerRecord<String, String> producerRecord = new ProducerRecord<>(
  paymentEvent.getCustomerId().toString() /* key */, 
  paymentEvent.toString() /* value */);

producerRecord.headers()
  .add("origin-cloud", "aws".getBytes(StandardCharsets.UTF_8)); 

producer.send(producerRecord);
```

## 注意事项
* 为了确保来自事件源的事件能够被[事件处理器](../event-processing/event-processor.md)正确读取，它们通常参考事件模式创建。事件模式通常在[Apache Avro](https://avro.apache.org/docs/current/spec.html)、[Protocol Buffers](https://developers.google.com/protocol-buffers)（Protobuf）或[JSON Schema](https://json-schema.org/)中定义。

* 对于基于云的架构，评估[CloudEvents](https://cloudevents.io/)的使用。CloudEvents提供标准化的[事件信封](../event/event-envelope.md)，包装事件，使常见的事件属性（如源、类型、时间和ID）普遍可访问，无论事件本身如何序列化。

* 在某些场景中，事件可能表示应该由读取事件的事件处理器执行的命令（指令、操作等）。有关详细信息，请参阅[命令](../event/command.md)模式。

## 参考资料
* 此模式部分源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Message.html)、[事件消息](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EventMessage.html)和[文档消息](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DocumentMessage.html)。
* [Apache Kafka 101: 介绍](/learn-kafka/apache-kafka/events/)提供了"什么是Kafka，它是如何工作的？"的入门知识，包括关于事件等核心概念的信息
