---
seo:
  title: 事件连接器
  description: 事件连接器通过将流与表或另一个事件流连接来增强事件流，提供查找数据。
---

# 事件连接器

[事件流](../event-stream/event-stream.md)可能需要与[表](../table/state-table.md)或另一个事件流连接（即丰富），以便提供关于其[事件](../event/event.md)的更全面的详细信息。

## 问题

我如何用额外的上下文丰富事件流或表？

## 解决方案

![event joiner](../img/event-joiner.svg)

我们可以通过在两个之间执行连接来将流中的事件与表或另一个事件流组合。连接基于原始事件流与其他事件流或表共享的键。我们还可以提供基于时间戳的窗口缓冲机制，以便当来自两个事件流的事件不是立即可用时，我们可以产生连接结果。另一种方法是将事件流与包含更静态数据的表连接，产生丰富的事件流。

## 实现

使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，我们可以从现有的Kafka主题创建持续更新的事件流（在此示例中，注意与数据仓库中的[事实表](https://en.wikipedia.org/wiki/Fact_table)的相似性）：

```sql
CREATE TABLE ratings (
    movie_id INT NOT NULL,
    rating FLOAT
);
```

然后我们可以从另一个变化较少的现有Kafka主题创建表。此表作为我们的参考数据（类似于数据仓库中的[维度表](https://en.wikipedia.org/wiki/Dimension_(data_warehouse))）。

```sql
CREATE TABLE movies (
    movie_id INT NOT NULL,
    title STRING,
    release_year INT  
);
```

要创建丰富事件的流，我们在事件流和表之间执行连接。

```sql
SELECT ratings.movie_id as id, title, rating
FROM ratings
LEFT JOIN movies ON ratings.movie_id = movies.movie_id;
```

## 注意事项

* 我们可以在事件流和表之间执行内连接或左外连接。
* 当两个或更多相应事件到达不同事件流或表时，连接对于启动后续处理也很有用。

## 参考资料

* [如何在Kafka Streams中连接流和查找表](https://developer.confluent.io/confluent-tutorials/joining-stream-table/kstreams/)
* [在Apache Flink® SQL中连接流和流](https://developer.confluent.io/confluent-tutorials/joining-stream-stream/flinksql/)
* [如何在Kafka Streams中连接表和表](https://developer.confluent.io/confluent-tutorials/joining-table-table/kstreams/)
