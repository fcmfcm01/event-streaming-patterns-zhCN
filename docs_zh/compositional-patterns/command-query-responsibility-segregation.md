---
seo:
  title: 命令查询职责分离
  description: 命令查询职责分离（CQRS）描述了数据更新和查询模型的分离。
---

# 命令查询职责分离 (CQRS)

数据库将数据的写入和读取混合在同一个地方：数据库。在某些情况下，最好将读取与写入分离。这样做有几个原因，但最主要的原因是应用程序现在可以以数据到达的确切形式保存数据，准确反映现实世界中发生的事情，同时以不同的形式读取数据，这种形式针对读取进行了优化。

例如，用户向购物车添加和移除商品的所有操作都将被记录为不可变事件的流：添加T恤、移除T恤等。然后将这些事件汇总到一个单独的服务于读取的视图中，例如汇总各种用户事件以表示购物车的准确内容。

## 问题

我们如何以数据到达的确切形式存储和保存数据，但从汇总和策划的视图中读取？

## 解决方案
![command-query-responsibility-segregation](../img/command-query-responsibility-segregation.svg)

将现实世界中发生的变化表示为[事件](../event/event.md)——订单已发货、行程已接受等——并将这些事件保留为记录系统。随后，将这些[事件](../event/event.md)聚合到一个视图中，该视图汇总事件以表示当前状态，允许应用程序查询当前值。

例如，账户的当前余额将是所有向账户添加或移除资金的支付事件的总和。记录系统是支付事件流。我们从中读取的视图将是账户余额。

## 实现

作为示例实现，您可以使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)在[事件流](../event-stream/event-stream.md)和[表](../table/state-table.md)中实现CQRS。

创建新的事件流很简单：

```sql
CREATE TABLE purchases (customer VARCHAR NOT NULL, item VARCHAR NOT NULL, qty INT);
```

可以使用熟悉的SQL语法直接插入[事件](../event/event.md)。

```sql
INSERT INTO purchases (customer, item, qty) VALUES
  ('jsmith', 'hats', 1),
  ('jsmith', 'hats', 1),
  ('jsmith', 'pants', 1),
  ('jsmith', 'sweaters', 1),
  ('jsmith', 'pants', 1),
  ('jsmith', 'pants', -1);
```

我们可以创建一个数据的汇总视图作为[表](../table/state-table.md)：

```sql  
CREATE TABLE customer_purchases AS
  SELECT customer, item, SUM(qty) AS total_qty
  FROM purchases
  GROUP BY customer, item;
```

并持续查询`customer_purchases`表状态的变化：

```sql 
SELECT * FROM customer_purchases;
```

## 注意事项

* CQRS比传统的简单[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)数据库实现增加了复杂性。

* 高性能应用程序可能受益于CQRS设计。隔离数据写入和读取的负载可能允许我们独立且适当地扩展这些方面。

* 微服务应用程序通常使用CQRS来扩展，为不同服务提供许多视图。相同的模式适用于地理分散的应用程序，如航班预订系统，这些系统在多个位置都是读取密集型的。

* 对CQRS系统的写入最终是一致的。写入不能立即读取，因为命令[事件](../event/event.md)的写入和查询模型的更新之间存在延迟。这可能为某些客户端应用程序带来复杂性，特别是在线服务。

## 参考资料

* 有关更多信息，请参阅Martin Fowler的[CQRS详细解释](https://martinfowler.com/bliki/CQRS.html)。
