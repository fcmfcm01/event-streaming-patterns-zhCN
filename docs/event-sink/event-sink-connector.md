---
seo:
  title: 事件接收器连接器
  description: 事件接收器连接器将事件从事件流传输到特定的外部系统
---

# 事件接收器连接器

将外部系统连接到[事件流平台](../event-stream/event-streaming-platform.md)允许高级和专门的集成。

## 问题

我们如何将应用程序或外部系统（如数据库）连接到[事件流平台](../event-stream/event-streaming-platform.md)，以便它可以接收[事件](../event/event.md)？

## 解决方案
![event-sink-connector](../img/event-sink-connector.svg)

事件接收器连接器是[事件接收器](event-sink.md)的特定实现。使用事件接收器连接器将[事件](../event/event.md)从[事件流](../event-stream/event-stream.md)传输到特定的外部系统。

## 实现

```
CREATE SINK CONNECTOR JDBC_SINK_POSTGRES_01 WITH (
    'connector.class'     = 'io.confluent.connect.jdbc.JdbcSinkConnector',
    'connection.url'      = 'jdbc:postgresql://postgres:5432/',
    'connection.user'     = 'postgres',
    'connection.password' = 'postgres',
    'topics'              = 'TEMPERATURE_READINGS_TIMESTAMP_MT',
    'auto.create'         = 'true',
    'auto.evolve'         = 'true'
);
```

当将关系数据库等系统连接到Apache Kafka®时，最常见的选项是使用[Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)。连接器从[事件流平台](../event-stream/event-streaming-platform.md)读取事件，执行任何必要的转换，并将[事件](../event/event.md)写入指定的[事件接收器](../event-sink/event-sink.md)。

## 注意事项

* 有许多事件接收器连接器可用于Apache Kafka，例如关系数据库或像AWS S3这样的对象存储系统的连接器。有关可用连接器，请参阅[Confluent Hub](https://www.confluent.io/hub/)。
* 安全策略以及法规合规性可能需要在事件接收器、事件接收器连接器和事件流平台之间进行适当的加密通信、身份验证和授权等设置。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[通道适配器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html)
