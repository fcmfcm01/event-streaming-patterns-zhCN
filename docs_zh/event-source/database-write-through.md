---
seo:
  title: 数据库直写
  description: 更新数据库中的值并创建具有至少一次保证的关联事件。
---

# 数据库直写

出于架构或遗留目的，以数据为中心的应用程序可能直接写入数据库。[事件处理应用程序](../event-processing/event-processing-application.md)需要使用[事件流](../event-stream/event-stream.md)上的[事件](../event/event.md)可靠地从这些系统消费数据。

## 问题

我们如何更新数据库中的值并创建具有至少一次保证的关联[事件](../event/event.md)？

## 解决方案
![db-write-through](../img/db-write-through.svg)

一些应用程序直接写入数据库表，因此数据库表是[事件源](event-source.md)。我们可以部署变更数据捕获（CDC）解决方案来持续捕获对该数据库表的写入（插入、更新、删除）并将它们作为[事件](../event/event.md)产生到[事件流](../event-stream/event-stream.md)上。然后流中的事件可以被[事件处理应用程序](../event-processing/event-processing-application.md)消费。此外，[事件流](../event-stream/event-stream.md)可以读入投影表，例如使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，以便其他应用程序可以查询它。

## 实现

[Kafka连接器](https://docs.confluent.io/platform/current/connect/index.html)提供了在Apache Kafka®和其他数据系统之间可扩展且可靠地移动数据的能力。有许多类型的连接器可以连接到许多类型的数据库。例如，使用Kafka Connect的[JDBC源连接器](https://docs.confluent.io/kafka-connect-jdbc/current/source-connector/index.html)允许将源数据库的变更流式传输到Kafka主题。

```bash
  {
    "connector.class": "MySqlCdcSource",
    "name": "MySqlCdcSourceConnector_0",
    "kafka.api.key": "****************",
    "kafka.api.secret": "******************************",
    "database.hostname": "database-2.<host-ID>.us-west-2.rds.amazonaws.com",
    "database.port": "3306",
    "database.user": "admin",
    "database.password": "**********",
    "database.server.name": "mysql",
    "database.whitelist": "employee",
    "table.includelist":"employees.departments",
    "snapshot.mode": "initial",
    "output.data.format": "AVRO",
    "tasks.max": "1"
  }
```

上面显示了部署MySQL源CDC连接器（Debezium）将MySQL数据库的数据流式传输到Kafka主题的示例配置。配置确定要连接的数据库（`database.hostname`）以及要源入Kafka的数据库元素（`table.includelist`）。`output.data.format`配置指示Kafka Connect在写入记录时使用哪种格式（在这种情况下是Apache Avro格式）。

[Confluent CLI](https://docs.confluent.io/confluent-cli/current/overview.html)可以从配置在命令行部署连接器，例如：
```bash
confluent connect create --config <connector-config-file.json>
```

## 注意事项

- 此模式是[事件源连接器](event-source-connector.md)的特化，它保证[事件源](../event-source/event-source.md)中表示的所有状态变更都在[事件流平台](../event-stream/event-streaming-platform.md)中被捕获。
- CDC数据流的处理保证（参见"保证交付"）选择——例如，至少一次、精确一次——取决于所使用的CDC和数据库技术。
- 源数据库表中的变更在CDC摄取的事件流中可用之前有一定的延迟。延迟量取决于各种因素，包括事件源连接器的功能和配置。在许多典型场景中，延迟少于几秒钟。
- [事件](../event/event.md)通常要求行键用作Kafka事件键（也称为记录/消息键），这是确保同一DB表行的所有[事件](../event/event.md)都进入同一Kafka主题分区的唯一方法，因此完全有序。它们通常还将删除建模为墓碑事件，即具有非空键和空值的事件。通过确保每行的完全有序事件，消费者看到每行这些事件的最终一致表示。

## 参考资料

* 有关Oracle DB的高级CDC连接器的详细信息，请参阅[Oracle CDC源连接器](https://www.confluent.io/blog/introducing-confluent-oracle-cdc-connector/)。
* 有关源连接器的信息，请参阅Confluent文档上的[将外部系统集成到Kafka](https://docs.confluent.io/cloud/current/connectors/index.html)。
* [Kafka Connect深度潜水 - JDBC源连接器](https://www.confluent.io/blog/kafka-connect-deep-dive-jdbc-source-connector/)博客文章
* 有关将数据库变更写入事件流的替代示例，请参阅[数据库旁路写入](database-write-aside.md)
* [Robin Moffatt](https://talks.rmoff.net/)的[不再有孤岛：集成数据库和Apache Kafka](https://rmoff.dev/no-more-silos)博客文章
