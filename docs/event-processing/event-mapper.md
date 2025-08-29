---
seo:
  title: 事件映射器
  description: 事件映射器在事件流平台中的域对象和事件之间移动数据。
---

# 事件映射器

处理静态数据的传统应用程序和处理动态数据的[事件处理应用程序](event-processing-application.md)可能需要通过[事件流平台](../event-stream/event-streaming-platform.md)共享数据。这些应用程序需要一个通用机制来将数据从[事件](../event/event.md)转换为域对象，反之亦然。

## 问题

如何在应用程序内部数据模型中的域对象和事件流平台中的事件之间移动数据，同时保持两者相互独立？

## 解决方案
![event-mapper](../img/event-mapper.svg)

事件映射器在传统应用程序和事件流平台之间提供独立性，使两者都不知道对方（理想情况下，两者甚至都不知道事件映射器本身）。

创建事件映射器，或使用现有的事件映射器，将[域模型](https://en.wikipedia.org/wiki/Domain_model)（或应用程序的内部数据模型）映射到事件流平台接受的数据格式，反之亦然。事件映射器读取域模型并将其转换为发送到事件流平台的传出事件。相反，事件映射器可用于基于传入事件创建或更新域对象。

## 实现

在此示例中，我们使用Apache Kafka®的Java生产者客户端来实现事件映射器，该映射器在事件写入[事件流](../event-stream/event-stream.md)（在Kafka中称为"主题"）之前从域模型对象（`Publication`）构造[事件](../event/event.md)（`PublicationEvent`）。

```java
private final IMapper domainToEventMapper = mapperFactory.buildMapper(Publication.class);
private final Producer<String, PublicationEvent> producer = ...

public void newPublication(String author, String title) {
  Publication newPub = new Publication(author, title);
  producer.send(author /* event key */, domainToEventMapper.map(newPub));
```

我们可以在第二个事件映射器中实现反向操作，将`PublicationEvent`事件转换回域对象更新：
```java
private final IMapper eventToDomainMapper = mapperFactory.buildMapper(Publication.class);
private final Consumer<String, PublicationEvent> consumer = ...

public void updatePublication(PublicationEvent pubEvent) {
  Publication newPub = eventToDomainMapper.map(pubEvent);
  domainStore.update(newPub);
```

## 注意事项

事件映射器可以选择性地验证转换对象的模式。有关详细信息，请参阅[模式验证器](../event-source/schema-validator.md)模式。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息映射器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingMapper.html)。
* 另请参阅[事件序列化器](../event/event-serializer.md)和[事件反序列化器](../event/event-deserializer.md)模式。
