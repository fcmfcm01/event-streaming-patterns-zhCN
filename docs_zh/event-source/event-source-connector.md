---
seo:
  title: 事件源连接器
  description: 事件源连接器模式用于将现有数据系统的数据集成到事件流平台中。
---

# 事件源连接器

[事件处理应用程序](../event-processing/event-processing-application.md)可能想要从现有数据系统消费数据，这些系统本身不是[事件源](event-source.md)。

## 问题

我们如何将云服务和传统系统（如关系数据库）连接到[事件流平台](../event-stream/event-streaming-platform.md)，将它们的静态数据转换为带有[事件](../event/event.md)的动态数据？

## 解决方案
![event-source-connector](../img/event-source-connector.svg)

一般来说，我们需要找到一种方法从源系统中提取数据作为[事件](../event/event.md)。例如，对于关系数据库，一种常见的技术是使用[变更数据捕获](https://en.wikipedia.org/wiki/Change_data_capture)，其中对数据库表的变更——如INSERT、UPDATE、DELETE——被捕获为[事件](../event/event.md)，然后可以摄取到另一个系统中。执行这种[事件](../event/event.md)提取和摄取的组件通常称为"连接器"。连接器将源系统转换为[事件源](../event-source/event-source.md)，然后从该数据生成[事件](../event/event.md)，最后将这些[事件](../event/event.md)发送到[事件流平台](../event-stream/event-streaming-platform.md)。

## 实现

当将云服务和传统系统连接到[Apache Kafka®](https://kafka.apache.org/)时，最常见的解决方案是使用[Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)。[Confluent Hub](https://www.confluent.io/hub/)上有数百个即用型连接器，包括像AWS S3这样的blob存储、像Salesforce和Snowflake这样的云服务、关系数据库、数据仓库、传统消息队列、平面文件等。Confluent还在云中提供了许多[完全托管的Kafka连接器](https://docs.confluent.io/cloud/current/connectors/index.html)。

## 注意事项

* 端到端数据交付保证（如精确一次交付或至少一次交付）主要取决于三个因素：（1）源事件源的能力，如云服务或关系数据库；（2）事件源连接器的能力；（3）目标事件流平台的能力，如Apache Kafka或Confluent。
* 安全策略以及法规合规性可能需要在事件源、事件源连接器和目标事件流平台之间进行适当的加密通信、身份验证和授权等设置。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[通道适配器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html)
