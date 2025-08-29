---
seo:
  title: 事件接收器
  description: 事件接收器从事件流平台读取事件，在概念上与事件源相反
---

# 事件接收器
[事件流平台](../event-stream/event-streaming-platform.md)中的各种组件将读取或接收[事件](../event/event.md)。事件接收器是这些组件的概括，可以包括[事件处理应用程序](../event-processing/event-processing-application.md)、云服务、数据库、物联网传感器、大型机等。

从概念上讲，事件接收器是[事件源](../event-source/event-source.md)的对立面。然而，在实践中，[事件处理应用程序](../event-processing/event-processing-application.md)等组件既可以作为[事件源](../event-source/event-source.md)，也可以作为事件接收器。

## 问题
我们如何在[事件流平台](../event-stream/event-streaming-platform.md)中读取（或消费/订阅）[事件](../event/event.md)？

## 解决方案

![event-sink](../img/event-sink.svg)

使用事件接收器，它通常作为[事件流平台](../event-stream/event-streaming-platform.md)中的客户端。例子包括[事件接收器连接器](event-sink-connector.md)（持续将[事件流](../event-stream/event-stream.md)从[事件流平台](../event-stream/event-streaming-platform.md)导出到云服务或关系数据库等外部系统）或[事件处理应用程序](../event-processing/event-processing-application.md)，如[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)应用程序或[Apache Flink®](https://nightlies.apache.org/flink/flink-docs-stable/)。

## 实现

Flink SQL示例：从现有的Apache Kafka®主题读取事件作为Flink表进行进一步处理。

```
CREATE TABLE orders (
    order_id INT,
    item_id INT,
    quantity INT,
    unit_price DOUBLE,
    ts TIMESTAMP(3),
    WATERMARK FOR ts AS ts
);
```

通用Kafka消费者应用程序：请参阅[Apache Kafka和Java入门](/get-started/java)以获取完整示例：

```
consumer.subscribe(Collections.singletonList("stream"));
      while (keepConsuming) { 
        final ConsumerRecords<String, EventRecord> consumerRecords = consumer.poll(Duration.ofSeconds(1));  
        recordsHandler.process(consumerRecords); 
      }
```

## 参考资料
* Apache Kafka的Kafka Streams库是开发人员实现读取、处理和写入事件的弹性应用程序和微服务的另一个流行选择。请参阅[过滤事件流](https://developer.confluent.io/confluent-tutorials/filtering/flinksql/)以获取第一个示例。
