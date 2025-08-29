---
seo:
  title: 事件源
  description: 事件源将事件写入事件流平台。
---

# 事件源
[事件流平台](../event-stream/event-streaming-platform.md)中的各种组件将生成[事件](../event/event.md)。事件源是这些组件的概括，可以包括[事件处理应用程序](../event-processing/event-processing-application.md)、云服务、数据库、物联网传感器、大型机等。

从概念上讲，事件源是[事件接收器](../event-sink/event-sink.md)的对立面。然而，在实践中，事件处理应用程序等组件既可以作为事件源，也可以作为事件接收器。

## 问题
我们如何在[事件流平台](../event-stream/event-streaming-platform.md)中创建[事件](../event/event.md)？

## 解决方案
![event-source](../img/event-source.svg)

使用事件源，它通常作为[事件流平台](../event-stream/event-streaming-platform.md)中的客户端。例子包括[事件源连接器](../event-source/event-source-connector.md)（持续将数据作为[事件流](../event-stream/event-stream.md)从云服务或关系数据库等外部系统导入到[事件流平台](../event-stream/event-streaming-platform.md)），或[事件处理应用程序](../event-processing/event-processing-application.md)，如[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)或[Apache Flink®](https://nightlies.apache.org/flink/flink-docs-stable/)应用程序。

## 实现
通常，Apache Kafka®的应用程序等实际组件会通过客户端库、API、网关等将[事件](../event/event.md)写入[事件流](../event-stream/event-stream.md)。我们也可以使用SQL语法直接写入事件；例如，[Flink SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)提供`INSERT`语句。
```
CREATE TABLE temperature_readings (
    sensor_id INT,
    temperature DOUBLE,
    ts TIMESTAMP(3),
    WATERMARK FOR ts AS ts
);
INSERT INTO temperature_readings VALUES
    (0, 55, TO_TIMESTAMP('2024-11-15 02:15:30')),
    (0, 50, TO_TIMESTAMP('2024-11-15 02:20:30')),
    (0, 45, TO_TIMESTAMP('2024-11-15 02:25:30'));
```

## 参考资料
* [如何构建客户端应用程序](https://docs.confluent.io/cloud/current/client-apps/index.html)以将事件写入[事件流](../event-stream/event-stream.md)。
