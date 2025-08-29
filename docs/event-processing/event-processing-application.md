---
seo:
  title: 事件处理应用程序
  description: 事件处理应用程序由一个或多个连接的事件处理器组成，形成处理拓扑以持续处理事件流和表中的数据。
---

# 事件处理应用程序
一旦我们的数据——如金融交易、货物跟踪信息、物联网传感器测量等——在事件流平台上作为事件流开始运动，我们就想利用这些数据并从中创造价值。[事件处理器](../event-processing/event-processor.md)是实现这一目标的构建块，但它们只解决用例的特定部分或步骤。

## 问题
我们如何为运动中的数据构建一个完整的应用程序，一个创建、读取、处理和/或查询[事件流](../event-stream/event-stream.md)以端到端解决用例的应用程序？

## 解决方案
![event-processing-application](../img/event-processing-application.svg)

我们通过将一个或多个[事件处理器](../event-processing/event-processor.md)组合成[事件流](../event-stream/event-stream.md)和[表](../table/state-table.md)的互连处理拓扑来构建事件处理应用程序。在这里，一个处理器的连续输出流是一个或多个下游处理器的连续输入流。应用程序的组合功能然后端到端地覆盖我们的用例，或至少覆盖我们想要的用例的尽可能多部分。（多少个应用程序应该实现一个用例是一个重要的设计决策，我们在这里不涉及。）组成更大事件处理应用程序的事件处理器通常是分布式的，在多个实例上运行，以允许大规模运动数据的弹性、并行、容错处理。

例如，应用程序可以从[事件流平台](../event-stream/event-streaming-platform.md)中的[事件存储](../event-storage/event-store.md)读取客户支付流，然后过滤某些客户的支付，然后按国家和按周聚合这些支付。处理模式是流处理；也就是说，数据24/7持续处理。一旦新的[事件](../event/event.md)可用，它们就被处理并通过[事件处理器](../event-processing/event-processor.md)的拓扑传播。

## 实现
Apache Kafka®是最受欢迎的[事件流平台](../event-stream/event-streaming-platform.md)。使用Kafka构建事件处理应用程序有几种选择。我们在这里介绍两种。

### Apache Flink®
Flink使开发人员能够使用SQL语法构建事件处理应用程序。

在下面的示例中，`movies`和`ratings`表由Kafka主题支持。
```sql
CREATE TABLE movies (
    movie_id INT NOT NULL,
    title STRING,
    release_year INT  
);

CREATE TABLE ratings (
    movie_id INT NOT NULL,
    rating FLOAT
);
```

正如我们所期望的，我们可以使用`INSERT`添加新的[事件](../event/event.md)：
```sql
INSERT INTO movies  VALUES (928, 'Dune: Part Two', 2024);
INSERT INTO ratings VALUES (928, 9.6);
```

我们还可以使用SQL语法执行流处理。在以下示例中，命令`CREATE TABLE .. AS SELECT ..`持续连接`ratings`和`movies`表以填充一个新表，该表包含关于被评分电影的元数据的评分。
```sql
CREATE TABLE rated_movies AS
    SELECT ratings.movie_id as id, title, rating
    FROM ratings
    LEFT JOIN movies ON ratings.movie_id = movies.movie_id;
```

### Kafka Streams

使用Apache Kafka的[Kafka Streams客户端库](https://docs.confluent.io/platform/current/streams/index.html)，我们可以在Java、Scala或其他JVM语言中实现事件处理应用程序。以下是类似于上面Flink示例的Kafka Streams示例：

```java
KStream<Integer, Rating> ratings = builder.table(<blabla>);
KTable<Integer, Movie> movies = builder.stream(<blabla>);
MovieRatingJoiner joiner = new MovieRatingJoiner();
KStream <Integer, EnrichedRating> enrichedRatings = ratings.join(movies, joiner);
```

请参阅教程[如何在Kafka Streams中连接流和查找表](https://developer.confluent.io/confluent-tutorials/joining-stream-table/kstreams/)以获取使用Kafka Streams构建事件处理应用程序的完整示例。

## 注意事项
* 在构建事件处理应用程序时，建议将应用程序限制在一个问题域内。虽然可以在应用程序中使用任意数量的事件处理器，但在大多数情况下，事件处理器应该密切相关（类似于如何设计微服务）。
* 事件处理应用程序本身也可以组合。这是实现事件驱动架构的常见设计模式，由应用程序和微服务舰队提供动力。在此设计中，一个应用程序的输出形成一个或多个下游应用程序的输入。这在概念上类似于[事件处理器](../event-processing/event-processor.md)的拓扑，如上所述。然而，实践中的一个关键区别是，不同的应用程序通常由组织内的不同团队构建。例如，由支付团队构建的面向客户的应用程序通过[事件流](../event-stream/event-stream.md)持续向反欺诈团队构建的应用程序和数据科学团队构建的另一个应用程序提供数据。

## 参考资料
* [事件流平台](../event-stream/event-streaming-platform.md)模式提供了事件处理应用程序如何在事件流平台中使用的更高级概述。
* 教程[如何使用Apache Flink® SQL连接两个数据流](https://developer.confluent.io/confluent-tutorials/joining-stream-stream/flinksql/)提供了使用SQL进行事件处理的逐步示例。
* 教程[如何使用Kafka Streams计算字段的总和](https://developer.confluent.io/confluent-tutorials/aggregating-sum/kstreams/)展示了如何在[事件流](../event-stream/event-stream.md)上应用聚合函数。
