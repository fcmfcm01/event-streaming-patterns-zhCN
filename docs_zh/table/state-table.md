---
seo:
  title: 状态表
  description: 状态表类似于关系数据库中的表，允许事件处理器记录和更新状态。
---

# 状态表

[事件处理器](../event-processing/event-processor.md)通常需要执行有状态操作，如聚合（例如，计算事件数量）。状态类似于关系数据库中的表，并且是可变的：它允许读写操作。事件处理器必须有一个高效且容错的状态管理机制——用于在处理输入事件时记录和更新状态——以确保计算的正确性并防止数据丢失和数据重复。

## 问题

事件处理器如何管理可变状态，类似于关系数据库中表的方式？

## 解决方案

![state-table](../img/state-table.svg)

我们需要实现一个可变状态表，允许事件处理器记录和更新状态。例如，要计算每个客户的支付数量，状态表提供客户（例如，客户ID）与当前支付计数之间的映射。

状态的存储后端可能因实现而异：选项包括本地状态存储（如RocksDB）、远程状态存储（如Amazon DynamoDB或NoSQL数据库）和内存缓存。通常建议使用本地状态存储，因为它们不会产生网络往返的额外延迟，这会提高事件处理器的端到端性能。

无论选择什么后端，状态表都应该是容错的，以确保强处理保证，如精确一次语义。容错可以通过例如将[事件源连接器](../event-source/event-source-connector.md)附加到状态表来执行变更数据捕获（CDC）来实现。这允许事件处理器持续将状态变更备份到[事件流](../event-stream/event-stream.md)中，并在故障或类似场景中恢复状态表。没有网络延迟需要处理。这也提供了一种在崩溃破坏本地存储后恢复状态的方法。

## 实现

[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)开箱即用地提供状态表。例如，我们可以通过将基础`movie_ticket_sales`表聚合到`movie_tickets_sold`表中来维护所有电影票销售的有状态计数：

```sql
CREATE TABLE movie_tickets_sold AS
    SELECT title,
           COUNT(total_ticket_value) AS tickets_sold
    FROM movie_ticket_sales
    GROUP BY title;
```

## 参考资料

* 另请参阅[投影表](../table/projection-table.md)模式。
