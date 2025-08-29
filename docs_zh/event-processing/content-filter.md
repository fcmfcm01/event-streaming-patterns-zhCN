---
seo:
  title: 内容过滤器
  description: 内容过滤器允许事件处理应用程序为特定用例定制事件，过滤掉不需要的字段，只保留最相关的信息。
---

# 内容过滤器

[事件处理应用程序](event-processing-application.md)中的[事件](../event/event.md)通常可能非常大。我们倾向于在数据到达时准确捕获数据，然后处理它，而不是先处理它并只存储结果。所以我们想要消费的事件通常包含比我们手头任务实际需要的更多信息。

例如，我们可能从第三方API拉取产品提要，并准确存储接收到的数据。后来，我们可能会问问题，"每个产品类别中有多少产品？"并发现每个事件包含100个字段，而我们真正只关心计数一个。至少，这是低效的；网络、内存和序列化成本比需要的要高100倍。但手动检查数据实际上变得痛苦——在100个字段中寻找并检查我们关心的那一个。

同样重要的是，我们可能有安全和数据隐私问题需要解决。想象一下，我们有一个表示用户个人详细信息和站点偏好的数据流。如果营销部门想要获得更多关于我们全球客户群的信息，我们可能能够共享用户的时区和货币设置，但_只有那些字段_。

我们需要一种存储完整事件的方法，同时只给消费者事件字段的子集。

## 问题

如何简单地从大型事件中只消费几个数据项？

## 解决方案

![content filter](../img/content-filter.svg)

创建一个[事件处理器](event-processor.md)，检查每个事件，提取感兴趣的字段，并向下游传递新的、较小的事件以进行进一步处理。

## 实现

作为示例，在[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)中，我们可以使用`SELECT`语句轻松地将丰富的事件流转换为更简单事件的流。

假设我们有一个名为`products`的表，其中每个事件包含大量字段。我们只对四个字段感兴趣：`producer_id`、`category`、`sku`和`price`。我们可以用以下查询将事件修剪为仅这些字段：

```sql
CREATE TABLE product_summaries AS
  SELECT
    product_id,
    category,
    sku,
    price
  FROM products;
```

我们可以使用Apache Kafka®客户端库[Kafka Streams](https://docs.confluent.io/platform/current/streams/index.html)执行等效转换，可能作为更大处理管道的一部分：

```java
builder.stream("products", Consumed.with(Serdes.Long(), productSerde))
    .mapValues(
        (product) -> {
          ProductSummary summary = new ProductSummary();

          summary.setCategory(product.getCategory());
          summary.setSku(product.getSku());
          summary.setPrice(product.getPrice());

          return summary;
        })
    .to("product_summaries", Produced.with(Serdes.Long(), productSummarySerde));
```

## 注意事项

由于过滤内容会创建新流，值得考虑新流将如何分区，如[分区并行性](../event-stream/partitioned-parallelism.md)模式中讨论的。默认情况下，新流将继承与其源相同的分区键，但我们可以重新分区数据以适应我们的新用例（例如，通过在Flink SQL中指定`DISTRIBUTED BY`子句）。

在上面的示例中，我们的第三方产品提要可能按供应商的唯一`product_id`分区，但对于此用例，按`category`分区过滤的事件可能更有意义。

有关详细信息，请参阅Flink的Kafka连接器接收器分区[文档](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/table/kafka/#sink-partitioning)。

## 参考资料

* 此模式源自Gregor Hohpe和Bobby Woolf的《企业集成模式》中的[内容过滤器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ContentFilter.html)。
* 对于从流中过滤掉整个事件，请考虑[事件过滤器](../event-processing/event-filter.md)模式。
