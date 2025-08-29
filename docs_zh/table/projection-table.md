---
seo:
  title: 投影表
  description: 投影表充当事件流或变更日志的物化视图，将事件分组和汇总为统一状态。
---

# 投影表

我们想要问事件流的第一个问题是，"我们现在在哪里？"

如果我们有销售事件流，我们希望手头有总销售数字。如果我们有`login`事件流，我们希望知道每个用户最后登录的时间。如果我们的卡车每分钟发送GPS数据，我们希望知道每辆卡车现在在哪里。

我们如何有效地汇总数据？我们如何保留完整的事件日志并享受"就地更新"风格数据库的快速查询？

## 问题

如何有效地汇总变更事件流以给出世界的当前状态？

## 解决方案
![Projection Table](../img/projection-table.svg)

我们可以维护一个行为就像传统数据库中物化视图的投影表。随着新事件的到来，表会自动更新，不断为我们提供系统的实时画面。具有相同键的事件被认为是相关的；较新的事件根据其内容被解释为对较旧事件的更新或删除。

与物化视图一样，投影表是只读的。要更改投影表，我们通过向表的基础流记录新事件来更改基础数据。

## 实现

[Apache Flink®](https://nightlies.apache.org/flink/flink-docs-stable/)支持[动态表](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/concepts/dynamic_tables/#dynamic-tables-amp-continuous-queries)作为其Table API和SQL支持的核心概念。Flink中动态表的连续查询与传统数据库中的物化视图非常相似。

作为示例，想象我们正在世界各地运送包裹。当包裹到达其旅程中的每个点时，都会记录其当前位置。

让我们从包裹签入事件流开始：

```sql
CREATE TABLE package_checkins (
    package_id INT,
    location STRING
);
```

要跟踪每个包裹的最新`location`：

```sql
CREATE TABLE current_package_locations AS
  SELECT
    package_id,
    LAST_VALUE(location) OVER w AS location
  FROM package_checkins
  WINDOW w AS (
    PARTITION BY package_id
    ORDER BY $rowtime
    ROWS BETWEEN UNBOUNDED PRECEDING
      AND CURRENT ROW
  );
```

随着新数据插入到`package_checkins`表中，`current_package_locations`表会更新，因此我们可以看到每个包裹的当前位置，而无需每次都扫描事件历史。

## 参考资料

* Flink SQL [窗口函数](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/hive-compatibility/hive-dialect/queries/window-functions/)
* 另请参阅[状态表](../table/state-table.md)模式。
