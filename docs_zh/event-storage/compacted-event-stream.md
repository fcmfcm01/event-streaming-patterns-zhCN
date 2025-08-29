---
seo:
  title: 压缩事件流
  description: 压缩事件流从事件流中删除表示过时信息并被新事件取代的事件
---

# 压缩事件流

[事件流](../event-stream/event-stream.md)通常表示状态的键控快照，类似于关系数据库中的[表](../table/state-table.md)。也就是说，[事件](../event/event.md)包含主键（标识符）和表示与事件相关的业务实体的最新信息的数据，例如每个客户账户的最新余额。[事件处理应用程序](../event-processing/event-processing-application.md)需要处理这些事件以确定业务实体的当前状态。但是，处理整个事件流历史通常不实用。

## 问题

如何将（键控的）[表](../table/state-table.md)永久存储在[事件流](../event-stream/event-stream.md)中，使用最少的空间？

## 解决方案

![compacted-event-stream](../img/compacted-event-stream.svg)

从流中删除表示过时信息并被新事件取代的事件。表的当前数据（即其最新状态）由流中的剩余事件表示。

这种方法将表的事件流的存储空间限制为`Θ(表中当前唯一键的数量)`，而不是`Θ(表变更事件的总数)`。在实践中，唯一键的数量（例如，唯一客户ID）通常比表变更的数量（例如，所有客户配置文件变更的总数）小得多。因此，压缩事件流在大多数情况下显著减少了存储空间。

## 实现

Apache Kafka®通过其[主题压缩](https://kafka.apache.org/documentation/#compaction)功能原生提供此功能。定期扫描流（Kafka中的主题）以删除已被具有相同键（如相同客户ID）的较新事件取代的任何旧事件。请注意，压缩是Kafka中的异步过程，因此压缩流可能包含一些被取代的事件，这些事件正在等待被压缩掉。

要使用Kafka创建名为`customer-profiles`的压缩事件流：

```bash
$ kafka-topics --create \
    --bootstrap-server <bootstrap-url> \
    --replication-factor 3 \
    --partitions 3 \
    --topic customer-profiles \
    --config cleanup.policy=compact

Created topic customer-profiles.
```

`kafka-topics`命令还可以验证当前主题的配置：

```bash
$ kafka-topics --bootstrap-server localhost:9092 --topic customer-profiles --describe

Topic: customer-profiles       PartitionCount: 3       ReplicationFactor: 1    Configs: cleanup.policy=compact,segment.bytes=1073741824
        Topic: customer-profiles       Partition: 0    Leader: 0       Replicas: 0     Isr: 0  Offline:
        Topic: customer-profiles       Partition: 1    Leader: 0       Replicas: 0     Isr: 0  Offline:
        Topic: customer-profiles       Partition: 2    Leader: 0       Replicas: 0     Isr: 0  Offline:
```

## 注意事项

压缩事件流允许一些优化：

* 首先，它们允许[事件流平台](../event-stream/event-streaming-platform.md)以数据特定的方式限制流的存储增长，而不是在预配置的时间段后普遍删除事件。
* 其次，拥有较小的流允许更快的恢复和系统迁移策略。

重要的是要理解，压缩故意通过删除如上定义的被取代事件来从事件流中删除历史数据。然而，在许多用例中，历史数据不应该被删除，例如对于金融交易流，其中每笔交易都需要被记录和存储。在这里，如果流存储是主要关注点，请使用[无限保留事件流](infinite-retention-event-stream.md)而不是压缩流。

## 参考资料

* 压缩事件流与[状态表](../table/state-table.md)模式高度相关。
* 压缩事件流的工作方式有点像简单的[日志结构化合并树](http://www.benstopford.com/2015/02/14/log-structured-merge-trees/)。
* Kafka主题的[清理策略配置](https://docs.confluent.io/platform/current/installation/configuration/topic-configs.html#topicconfigs_cleanup.policy)。
