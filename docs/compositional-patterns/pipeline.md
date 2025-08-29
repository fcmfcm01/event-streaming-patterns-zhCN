---
seo:
  title: 管道
  description: 通过一系列独立的处理阶段对事件流或表中的一系列事件执行复杂操作。
---

# 管道

单个[事件流](../event-stream/event-stream.md)或[表](../table/state-table.md)可以被多个[事件处理应用程序](../event-processing/event-processing-application.md)使用，其[事件](../event/event.md)可能在此过程中经过多个处理阶段（例如，过滤器、转换、连接、聚合）来实现更复杂的用例。

## 问题

如何通过一系列独立的处理阶段来实现事件流和/或表的单个处理目标？

## 解决方案
![pipeline](../img/pipeline.svg)

我们可以通过[事件处理应用程序](../event-processing/event-processing-application.md)在[事件流平台](../event-stream/event-streaming-platform.md)中组合[事件流](../event-stream/event-stream.md)和[表](../table/state-table.md)，创建[事件处理器](../event-processing/event-processor.md)的管道——也称为拓扑——持续处理流经它们的事件。在这里，一个处理器的输出是一个或多个下游处理器的输入。管道，特别是为流式[ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load)等用例创建的管道，可能包括[事件源连接器](../event-source/event-source-connector.md)和[事件接收器连接器](../event-sink/event-sink-connector.md)，它们分别持续从/向外部服务和系统导入和导出数据作为流。连接器对于将此类系统中的静态数据转换为动态数据特别有用。

退一步说，我们可以看到事件流平台中的管道帮助公司为动态数据构建"中央神经系统"。

## 实现

作为示例，我们可以使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)通过一系列处理阶段运行事件流，从而创建一个持续处理动态数据的管道。

```sql
CREATE TABLE orders ( 
  customer_id INTEGER NOT NULL,
  order_id STRING NOT NULL,
  item_id STRING,
  price DOUBLE
);
```

我们还将创建一个（持续更新的）客户表，其中包含每个客户的最新配置文件信息，例如他们当前的家庭地址。

```sql
CREATE TABLE customers (
  customer_id INTEGER NOT NULL,
  customer_name STRING,
  address STRING
);
```

接下来，我们通过将订单流与我们的客户表连接来创建新流：

```sql
CREATE TABLE orders_enriched AS
  SELECT o.customer_id AS cust_id, o.order_id, o.price, c.customer_name, c.address
  FROM orders o
  LEFT JOIN customers c 
  ON o.customer_id = c.customer_id;
```

接下来，我们创建一个流，通过聚合订单中单个项目的价格来为每个订单添加订单总额：

```sql
CREATE TABLE orders_with_totals AS
  SELECT cust_id, order_id, sum(price) AS total 
  FROM orders_enriched
  GROUP BY cust_id, order_id;
```

## 注意事项

* 同一事件流或表可以参与多个管道。因为流和表是持久存储的，应用程序在处理相应数据的方式和时间上有很大的灵活性，并且它们可以相互独立地这样做。
* 管道中的各种处理阶段创建自己的派生流/表（如上面Flink SQL示例中的`orders_enriched`表），这些流/表又可以作为其他管道和应用程序的输入。这允许在整个组织中对事件进行进一步和更复杂的组合和重用。

## 参考资料

此模式受到Gregor Hohpe和Bobby Woolf的《企业集成模式》中[管道和过滤器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html)的影响。但是，它更强大和灵活，因为它使用[事件流](../event-stream/event-stream.md)作为管道。
