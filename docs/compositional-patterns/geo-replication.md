---
seo:
  title: 地理复制
  description: 地理复制使多个事件流平台能够连接，使得在一个站点可用的事件在其他站点也可用
---

# 地理复制

许多架构都有跨越[事件流平台](../event-stream/event-streaming-platform.md)、数据中心或地理区域边界部署在多个数据中心的事件流。在这些情况下，一个事件流平台中的客户端应用程序能够访问在另一个平台中产生的[事件](../event/event.md)可能是有用的。所有客户端不应该被迫从源事件流平台读取，这可能会产生高延迟和数据出口成本。相反，采用一次移动多次读取的方法，数据可以复制到本地数据中心，客户端可以在那里快速且廉价地进行所有处理。

## 问题

如何连接多个[事件流平台](../event-stream/event-streaming-platform.md)，使得在一个站点可用的事件在其他站点也可用？

## 解决方案
![geo-replication](../img/geo-replication.svg)

在两个[事件流平台](../event-stream/event-streaming-platform.md)之间创建连接，使目标平台能够从源平台读取。理想情况下，这是实时完成的，这样当新事件在源事件流平台中发布时，它们可以立即逐字节复制到目标事件流平台。这允许目标中的客户端应用程序利用相同的数据集。

## 实现

通常，在实践中并非所有事件流都启用复制。总是有例外、组织限制、技术约束或其他原因，我们不想复制绝对所有内容。相反，我们可以基于每个流进行此操作，在那里我们可以将源流映射到目标流。

使用Apache Kafka®，我们可以通过以下几种方式之一来实现这一点。

### 选项1：集群链接

[集群链接](https://docs.confluent.io/cloud/current/multi-cloud/cluster-linking.html)使事件流平台之间的数据共享变得容易，在它们之间镜像Kafka主题（即流）。因为集群链接使用原生复制协议，客户端应用程序在灾难恢复场景中可以轻松进行故障转移。

```sh
confluent kafka link create east-west ...
confluent kafka mirror create <destination topic> --link east-west ...
```

其他消息系统如RabbitMQ和ActiveMQ提供类似功能，但没有相同级别的并行性。

### 选项2：基于Connect的复制

操作员可以使用Confluent的[复制器](https://docs.confluent.io/cloud/current/clusters/migrate-topics-on-cloud-clusters.html)或Kafka的[MirrorMaker](https://kafka.apache.org/documentation/#georeplication)（版本2）设置集群间数据流，这些工具在不同Kafka环境之间复制数据。与集群链接不同，这些是基于Kafka Connect构建的单独服务，具有内置的生产者和消费者。

## 注意事项

请注意，事件流平台之间的这种复制是异步的，这意味着在源中记录的事件可能不会立即在目标中可用。事件流平台之间也有同步复制（例如[多区域集群](https://docs.confluent.io/platform/current/multi-dc-deployments/index.html)），但这通常仅限于事件流平台在同一操作域内的情况。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息桥接](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingBridge.html)
