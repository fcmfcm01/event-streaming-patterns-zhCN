---
seo:
  title: 相关性标识符
  description: 相关性标识符在事件流平台（如Apache Kafka®）上实现请求响应协议。
---

# 相关性标识符
[事件处理应用程序](../event-processing/event-processing-application.md)可能想要实现[事件协作](../compositional-patterns/event-collaboration.md)，其中[事件](../event/event.md)用于传输请求和响应。当应用程序通过事件协作时，它们需要一种方法来关联特定请求的事件响应数据。

## 问题
当应用程序请求信息并收到响应时，应用程序如何知道哪个请求对应于该特定响应？

## 解决方案
![correlation-identifier](../img/correlation-identifier.svg)

[事件处理器](../event-processing/event-processor.md)生成一个事件，作为请求。在发送请求事件之前，向其中添加一个全局唯一标识符。这允许响应的事件处理器在响应事件中包含该标识符，以便请求处理器可以关联请求和响应。

## 实现
在Kafka中，我们可以在生产请求事件时向Kafka记录头部添加全局唯一标识符。以下示例使用Kafka Java生产者客户端。
```java
ProducerRecord<String, String> requestEvent = new ProducerRecord<>("request-event-key", "request-event-value"); 
requestEvent.headers().add("requestID", UUID.randomUUID().toString());
requestEvent.send(producerRecord);
```

在响应的事件处理器中，我们首先从请求事件中提取相关性标识符（这里，标识符称为`requestID`），然后将该标识符添加到响应事件中。
```Java
ProducerRecord<String, String> responseEvent = new ProducerRecord<>("response-event-key", "response-event-value"); 
responseEvent.headers().add("requestID", requestEvent.headers().lastHeader("requestID").value());
responseEvent.send(producerRecord);
```

## 参考资料
* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[相关性标识符](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html)。
* 相关性标识符可以用作[事件协作](../compositional-patterns/event-collaboration.md)的一部分，这是一种分散的事件处理应用程序协作实现分布式工作流解决方案的模式。
* 标记请求及其关联响应的想法存在于许多其他协议中。例如，通过IMAP连接的电子邮件客户端将发送以唯一ID为前缀的命令（通常是`a001`、`a002`等），服务器将异步响应，用匹配的ID标记其响应。
