---
seo:
   title: 声明检查
   description: 如果事件流平台对事件有自然或配置的大小限制，不要存储整个事件，只存储对事件的引用
---

# 声明检查

有时压缩可以减少消息大小，但各种用例涉及大型消息负载，其中压缩可能不够。
这些用例通常与图像、视频或音频处理相关：图像识别、视频分析、音频分析等。

## 问题

我们如何处理[事件](../event/event.md)负载太大或太昂贵而无法通过[事件流平台](../event-stream/event-streaming-platform.md)的用例？

## 解决方案

![claim-check](../img/claim-check.svg)

不要将整个事件存储在事件流平台中，而是将事件负载存储在可以在生产者和消费者之间共享的持久外部存储中。
生产者可以将引用地址写入事件流平台，下游客户端使用该地址从外部存储检索事件，然后根据需要处理它。

## 实现

存储在Kafka中的事件只包含对外部存储中对象的引用。
这可以是完整的URI字符串、具有单独字段（如存储桶名称和文件名）的[抽象数据类型](https://en.wikipedia.org/wiki/Abstract_data_type)（例如，Java对象），或识别对象所需的任何字段。可选地，事件可能包含额外的数据字段以更好地描述对象（例如，元数据，如谁创建了对象）。

以下示例使用Kafka的Java生产者客户端。在这里，我们保持简单，因为事件的值除了对其各自对象在外部存储中的引用（URI）之外不存储任何信息。

```java
  // 将对象写入外部存储
  storageClient.putObject(bucketName, objectName, object);

  // 将URI写入Kafka
  URI eventValue = new URI(bucketName, objectName);
  producer.send(new ProducerRecord<String, URI>(topic, eventKey, eventValue));
```

## 注意事项

[事件源](../event-source/event-source.md)负责确保数据正确存储在外部存储中，使得[事件](../event/event.md)中传递的引用有效。
由于生产者应该原子地执行此操作，请考虑与[数据库旁路写入](../event-source/database-write-aside.md)中提到的相同问题。

另外，如果使用[压缩事件流](../event-storage/compacted-event-stream.md)来存储"引用"事件（例如，在Kafka的情况下进行主题压缩），那么压缩将只删除带有引用的事件。但是，它不会从外部存储中删除被引用的（大型）对象本身，因此该对象需要不同的过期机制。

## 参考资料

* 此模式在思想上类似于Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[声明检查](https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html)
* 处理大型消息的替代方法是[事件分块](../event-processing/event-chunking.md)
