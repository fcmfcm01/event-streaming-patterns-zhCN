---
seo:
  title: 事件信封
  description: 事件信封允许具有不同数据格式的应用程序在事件流平台上进行通信。
---

# 事件信封
[事件流平台](../event-stream/event-streaming-platform.md)允许许多不同类型的应用程序协同工作。事件信封为通过[事件流应用程序](../event-processing/event-processing-application.md)发送的所有[事件](../event/event.md)提供了一组标准的已知字段。信封独立于底层事件格式，通常引用加密类型、模式、键和序列化格式等属性。信封类似于网络中的协议头部（例如，TCP-IP）。

## 问题
我如何向[事件流平台](../event-stream/event-streaming-platform.md)中的所有参与者传达信息，而不依赖于事件负载？例如，我如何传达如何解密[事件](../event/event.md)、使用什么模式，或者什么ID定义事件的唯一性？

## 解决方案
![event-envelope](../img/event-envelope.svg)

使用事件信封，使用[事件流平台](../event-stream/event-streaming-platform.md)的所有参与者（或更广泛地）同意的标准格式包装事件数据。[Cloud Events](https://cloudevents.io/)——标准化对ID、模式、键和其他常见事件属性的访问——是事件信封模式的行业标准示例。

## 示例实现
使用基本的Java消费者和生产者，您可以使用辅助函数将应用程序的不可变负载包装成符合[事件流平台](../event-stream/event-streaming-platform.md)预期格式的信封：

```java
static <T> Envelope<T> wrap(T payload, Iterable<Header> headers) {
	return new Envelope(serializer(payload), headers);
}
static <T> T unwrap(Envelope<T> envelope) {
	return envelope.payload;
}
```

## 参考资料
* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[信封包装器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/EnvelopeWrapper.html)。
* [如何在严格和动态模式之间选择](https://www.confluent.io/blog/spring-kafka-protobuf-part-1-event-data-modeling/)
* 参见[CloudEvents](https://cloudevents.io/)了解以通用方式描述事件头部信息的规范。
