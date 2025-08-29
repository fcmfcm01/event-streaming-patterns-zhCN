---
seo:
  title: 事件流API
  description: 事件流API为应用程序提供了一种一致且可靠的方式来连接到事件流平台。
---

# 事件流API

连接到[事件流平台](../event-stream/event-streaming-platform.md)的应用程序需要以一致且可靠的方式进行连接。

## 问题

我的应用程序如何连接到事件流平台来发送和接收[事件](../event/event.md)？

## 解决方案
![event-streaming-api](../img/event-streaming-api.svg)

事件流平台提供了一个应用程序编程接口（API），允许应用程序在平台上可靠地通信。API提供了一个逻辑清晰且文档完善的协议，定义了消息结构和数据交换方法。高级库实现了这些协议，因此可以使用各种技术和编程语言与平台交互。这些高级库允许应用程序专注于业务逻辑，而将平台通信的细节留给API处理。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息端点](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageEndpoint.html)。
* [Kafka协议指南](https://kafka.apache.org/protocol.html)提供了关于Kafka中实现的线协议的详细信息。
* [Kafka API文档](https://kafka.apache.org/documentation/#api)包含有关用于读取、写入和管理Kafka的各种API的信息。
