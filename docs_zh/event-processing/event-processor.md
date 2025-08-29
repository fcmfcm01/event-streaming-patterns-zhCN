---
seo:
  title: 事件处理器
  description: 事件处理器对事件应用特定的处理操作。事件处理器通常被更大的事件处理应用程序使用和组合。
---

# 事件处理器
一旦我们的数据——如金融交易、货物跟踪信息、物联网传感器测量等——在[事件流平台](../event-stream/event-streaming-platform.md)上作为[事件流](../event-stream/event-stream.md)开始运动，我们就想利用它并从中创造价值。我们如何做到这一点？

## 问题
我们如何在[事件流平台](../event-stream/event-streaming-platform.md)中处理[事件](../event/event.md)？

## 解决方案
![event-processor](../img/event-processor.svg)
我们构建一个事件处理器，这是一个读取[事件](../event/event.md)并处理它们的组件，可能作为其处理结果写入新事件。事件处理器可能作为[事件源](../event-source/event-source.md)和/或[事件接收器](../event-sink/event-sink.md)行动；在实践中，两者都很常见。事件处理器可以是分布式的，这意味着它在不同机器上有多个实例运行。在这种情况下，事件的处理在这些实例之间并发发生。

事件处理器的一个重要特征是它应该允许与其他事件处理器组合。在实践中，我们很少单独使用单个事件处理器。相反，我们在[事件处理应用程序](event-processing-application.md)中组合和连接一个或多个事件处理器（通过[事件流](../event-stream/event-stream.md)），该应用程序完全端到端地实现一个特定用例，或实现整体业务逻辑的一个子集，限于特定域的有界上下文（例如，在微服务架构中）。

事件处理器在事件处理应用程序中执行特定任务。将事件处理器视为更大处理拓扑中的一个处理节点或处理步骤。例子包括将事件类型映射到域对象、从[事件流](../event-stream/event-stream.md)中过滤出重要事件、通过将其连接到另一个流或数据库表来丰富事件流，触发警报，或创建供其他应用程序消费的新事件。

## 实现

有多种方法可以使用事件处理器创建[事件处理应用程序](../event-processing/event-processing-application.md)。我们将看两种。

#### Apache Flink® SQL

[Flink SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)为创建事件处理应用程序提供熟悉的标准SQL语法。Flink SQL解析SQL命令并构建和管理我们定义为事件处理应用程序一部分的事件处理器。

在以下示例中，我们创建一个Flink SQL查询，从`readings`事件流读取数据并"清理"事件值。查询将清理的读数发布到名为`clean_readings`的新表。在这里，此查询作为包含多个互连事件处理器的事件处理应用程序。

```sql
CREATE TABLE clean_readings AS
    SELECT sensor,
           reading,
           UPPER(location) AS location
    FROM readings;
```

使用Flink SQL，我们可以将命令的每个部分视为不同事件处理器的构建：

* `CREATE TABLE`定义此应用程序将生产事件的新输出[事件流](../event-stream/event-stream.md)。
* `SELECT ...`是一个映射函数，接受每个输入事件并按定义"清理"它。在此示例中，清理只是意味着将每个输入读数中的`location`字段大写。
* `FROM ...`是一个源事件处理器，定义整个应用程序的输入事件流。

#### Kafka Streams
[Kafka Streams DSL](https://docs.confluent.io/platform/current/streams/developer-guide/dsl-api.html)为[事件流](../event-stream/event-stream.md)和[表](../table/state-table.md)提供抽象，以及有状态和无状态转换函数（`map`、`filter`等）。这些函数中的每一个都可以作为我们用Kafka Streams库构建的更大[事件处理应用程序](../event-processing/event-processing-application.md)中的事件处理器。

```java
builder
  .stream("readings");
  .mapValues((key, value)-> 
    new Reading(value.sensor, value.reading, value.location.toUpperCase()) 
  .to("clean");
```

在上面的示例中，我们使用[Kafka Streams `StreamsBuilder`](https://kafka.apache.org/38/javadoc/org/apache/kafka/streams/StreamsBuilder.html)构建流处理拓扑。

* 首先，我们使用`stream`函数创建输入流。这从指定的Apache Kafka®主题创建事件流。
* 接下来，我们使用`mapValues`函数转换事件。此函数接受一个事件并返回一个新事件，对原始事件的值进行任何所需的转换。
* 最后，我们使用`to`函数将转换后的事件写入目标Kafka主题。此函数终止我们的流处理拓扑。

## 注意事项

* 虽然构建"多用途"事件处理器可能很诱人，但以可组合的方式设计处理器很重要。通过将处理器构建为离散单元，更容易推理每个处理器做什么，进而推理事件处理应用程序做什么。

## 参考资料
* 参见[事件处理应用程序](../event-processing/event-processor.md)模式。事件处理应用程序由事件处理器组成。
* 在[Kafka Streams](https://kafka.apache.org/38/documentation/streams/core-concepts#streams_topology)中，处理器是处理器拓扑中的一个节点，代表转换事件的步骤。
* 博客文章[Apache Kafka介绍：Kafka如何工作](https://www.confluent.io/blog/apache-kafka-intro-how-kafka-works/)提供了关于核心Kafka概念的详细信息，如事件和主题。
