---
seo:
  title: 事件分割器
  description: 事件分割器将一个事件分割成多个可以以不同方式处理的事件。
---

# 事件分割器

一个[事件](../event/event.md)可能实际上包含多个子事件，每个子事件可能需要以不同的方式处理。

## 问题

如何将[事件](../event/event.md)分割成多个事件以进行不同的处理？

## 解决方案
![event-splitter](../img/event-splitter.svg)

将原始事件分割成多个子事件。
然后为每个子事件发布一个事件。

## 实现

许多事件处理技术支持此操作。[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)支持通过`UNNEST`函数将数组展开为多个事件。下面的示例处理每个输入事件，展开数组并为每个元素生成新事件。

```sql
CREATE TABLE orders (
    order_id INT NOT NULL,
    tags ARRAY<STRING>
);
```

```sql
CREATE TABLE exploded_orders AS
  SELECT order_id, tag
  FROM orders
  CROSS JOIN UNNEST(tags) AS t (tag);
```

Apache Kafka®客户端库[Kafka Streams](https://kafka.apache.org/documentation/streams/)有一个类似的方法，称为`flatMap()`。
下面的示例处理每个输入事件并生成具有新键和值的新事件。

```java
KStream<Long, String> myStream = ...;
KStream<String, Integer> splitStream = myStream.flatMap(
    (eventKey, eventValue) -> {
      List<KeyValue<String, Integer>> result = new LinkedList<>();
      result.add(KeyValue.pair(eventValue.toUpperCase(), 1000));
      result.add(KeyValue.pair(eventValue.toLowerCase(), 9000));
      return result;
    }
  );
```

或者，正如我祖母常说的：

> _从前有个人来自曼哈顿，_  
> _他需要扁平化的事件。_  
> _他想出了一个方案_  
> _在`stream`上调用`flatMap`，_  
> _然后把它写下来作为一个模式。_

## 注意事项

* 如果您有必须路由到不同事件流的子事件，请参阅[事件路由器](../event-processing/event-router.md)模式，用于将事件路由到不同位置。
* 对于容量规划和大小调整，请考虑将原始事件分割成N个子事件会导致写入放大，增加[事件流平台](../event-stream/event-streaming-platform.md)必须管理的事件量。
* 用例可能要求您跟踪父事件和子事件的谱系。如果是这样，请确保子事件包含一个数据字段，其中包含对原始父事件的引用（例如，唯一标识符）。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[分割器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html)。
