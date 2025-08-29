---
seo:
  title: 事件过滤器
  description: 事件过滤器允许事件处理应用程序在事件流中的事件子集上操作。
---

# 事件过滤器

[事件处理应用程序](event-processing-application.md)可能需要在[事件流](../event-stream/event-stream.md)中的[事件](../event/event.md)子集上操作。

## 问题

应用程序如何从事件流中仅选择相关事件（或丢弃无趣的事件）？

## 解决方案
![event-filter](../img/event-filter.svg)

## 实现

作为示例，[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)让我们使用熟悉的SQL语法创建过滤的事件流：

```sql
CREATE TABLE payments_only AS
    SELECT *
      FROM all_transactions
      WHERE type = 'purchase';
```

Apache Kafka®的[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)客户端库在其DSL中提供了`filter`操作符。此操作符过滤掉不匹配给定谓词的事件：

```java
builder
  .stream("transactions-topic")
  .filter((key, transaction) -> transaction.type == "purchase")
  .to("payments-topic");
```

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[消息过滤器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Filter.html)。
* 有关过滤事件流的详细示例，请参阅教程[如何使用Apache Flink® SQL过滤事件流](https://developer.confluent.io/confluent-tutorials/filtering/flinksql/)。
