---
seo:
  title: 命令
  description: 命令是演进中的事件流架构中的常见模式，作为指令发布给特定接收者执行任务。命令可能表明进一步解耦和职责分离的机会。
---

# 命令

_事件_涉及事实——用户发送他们的新地址，产品离开仓库——我们首先记录这些事实，而不立即考虑接下来会发生什么。

相比之下，_命令_用于调用特定操作——用户点击`[购买]`按钮——系统执行该操作（例如，通过触发订单处理）。

## 问题

我们如何使用[事件流平台](../event-stream/event-streaming-platform.md)来调用另一个应用程序中的过程？

## 解决方案
![Command](../img/command-event1.svg)

一个服务可以通过使用[事件流](../event-stream/event-stream.md)来调用另一个服务中的函数调用。生产者服务（应用程序A）创建一个命令，详细说明消费者服务（应用程序B）要执行的操作。命令还包括执行命令所需的任何必要支持信息。

命令通常在没有期望响应的情况下发布，除了[事件代理](../event-stream/event-broker.md)的简单接收确认。表示命令的实际记录被生产到[事件流](../event-stream/event-stream.md)中，并以强持久性保证存储。写入者假设命令将被适当的订阅者服务正确处理，监控和错误处理的责任落在系统的其他地方。这与Actor模型非常相似：

1. Actor有一个收件箱。
2. 客户端向收件箱写入消息。
3. Actor在适当的时候处理消息。

如果生产者需要明确的响应值，消费服务可以将结果记录写回第二个事件流。发布命令与其响应记录的相关性通常使用[相关性标识符](../event/correlation-identifier.md)执行。

## 实现

假设我们有一个`[购买]`按钮，应该触发我们仓储系统中的`dispatchProduct(12005)`函数调用。我们可以拆分调用，而不是直接调用函数。我们创建一个命令流：

```sql
CREATE STREAM dispatch_products (
  order_id BIGINT KEY,
  address VARCHAR
) WITH (
  KAFKA_TOPIC = ' dispatch_products',
  VALUE_FORMAT = 'AVRO',
  PARTITIONS = 2
);
```

我们启动一个向流中插入数据的过程：

```sql
INSERT INTO dispatch_products ( order_id, address ) VALUES ( 12004, '1 Streetford Road' );
INSERT INTO dispatch_products ( order_id, address ) VALUES ( 12005, '2 Roadford Avenue' );
INSERT INTO dispatch_products ( order_id, address ) VALUES ( 12006, '3 Avenue Fordstreet' );
```

最后，我们启动第二个过程，监视事件流并为每个事件调用`dispatchProduct`过程：

```java
    ...
    Serde<GenericRecord> valueGenericAvroSerde = ...
    StreamsBuilder builder = new StreamsBuilder();

    KStream<Long, GenericRecord> dispatchStream = builder.stream(
      "dispatch_products",
      Consumed.with(Serdes.Long(), valueGenericAvroSerde)
    );

    dispatchStream.foreach((key, value) -> warehouse.dispatchProduct(key, value));
```

## 注意事项

这种方法有效，但可能表明错过了改进系统整体架构的机会。

考虑当我们需要更多操作时会发生什么。假设`[购买]`还应该触发向客户发送电子邮件和短信通知。仓储软件应该完成其工作，然后向两个新主题写入`SendEmail`和`SendText`命令吗？或者这两个新事件应该由写入`DispatchProduct`的同一过程写入？

一个月后，当我们需要销售数据时，我们应该计算已分发的产品数量，还是发送的电子邮件数量？我们应该计算两者来验证它们是否一致吗？系统增长了一点，我们必须问："`[购买]`按钮后面有多少代码？发布周期是什么？改变它是否成为阻碍？"`[购买]`按钮对整个公司很重要，这是正确的，但它的维护不应该让公司受到勒索。

这里的根本问题是，在从单体中的函数调用转向向特定接收者发布特定命令的系统时，我们解耦了函数调用，但_没有_解耦底层概念。当我们这样做时，架构会以成长的痛苦<sup>1</sup>来回应。

更好的解决方案是意识到我们的命令实际上是两个交织在一起的概念："发生了什么？"和"谁需要知道？"

通过将这些概念分开，我们可以清理我们的架构。我们允许一个过程专注于记录发生的事情的事实，而其他过程自己决定是否关心这些事实。当`[购买]`点击发生时，我们应该只写入一个`Order`事件。然后仓储、通知和销售可以选择做出反应，无需任何协调。

![Command](../img/command-event2.svg)

简而言之，命令与单一受众紧密耦合，而事件只是一个解耦的事实，可供任何感兴趣的人使用。命令本身不是一个坏模式，但它们是强耦合的指标，可能表明进一步解耦的机会。

以这种方式看待系统需要稍微改变视角——一种建模我们过程的新方式——并为更容易协作但实际上承担更少个人责任的系统开辟了机会。

## 参考资料

* 如果存在函数链，其中一个的结果被输入到下一个的参数中，这种方法可能会变得复杂。在这种情况下，考虑使用[事件协作](../compositional-patterns/event-collaboration.md)。
* 参见[设计事件驱动系统](https://www.confluent.io/designing-event-driven-systems/)——特别是第5章"事件：协作的基础"——进行进一步讨论。
* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[命令消息](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CommandMessage.html)。

## 脚注

<sup>1</sup> _在这一点上，团队中的某人会说："我们直接调用函数会更好。"如果我们停在那里，他们会有合理的观点。_
