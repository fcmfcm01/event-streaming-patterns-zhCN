---
seo:
  title: 等待N个事件
  description: 应用程序可以等待触发处理，直到事件流接收到设定数量的事件。
---

# 等待N个事件

有时[事件](../event/event.md)在发生几次后才变得重要。

用户可以尝试登录五次，但之后我们将锁定他们的账户。包裹递送将尝试三次，然后我们要求客户从仓库收集它。游戏玩家在杀死第一百个Blarg后获得奖杯。

我们如何有效地监视逻辑上相似的事件？

## 问题

应用程序如何等待一定数量的事件发生后再执行处理？

## 解决方案
![wait for N events](../img/wait-for-n-events.svg)

要将相关事件作为一个组考虑，我们需要按给定键对它们进行分组，然后计算该键的出现次数。

## 实现

使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)，我们可以轻松创建一个[投影表](../table/projection-table.md)，按特定键对事件进行分组和计数。

作为示例，想象我们正在处理非常大的金融交易。我们只想在两名经理审查和批准这些交易后处理它们。

我们将从经理签名事件表开始：

```sql
CREATE TABLE trade_reviews (
  trade_id BIGINT,
  manager_id VARCHAR,
  signature VARCHAR,
  approved BOOLEAN
);
```

我们将按`trade_id`对审查进行分组，然后`COUNT()`我们为每个看到的批准数量（`approved = TRUE`），只保留至少有两个的（`HAVING COUNT(*) >= 2`）：

```sql
CREATE TABLE approved_trades AS
    SELECT trade_id, COUNT(*) AS approvals
    FROM trade_reviews
    WHERE approved = TRUE
    GROUP BY trade_id
    HAVING COUNT(*) >= 2;
```

在一个终端中查询该表：

```
SELECT * FROM approved_trades;
```

在另一个终端中插入一些数据：

```sql
INSERT INTO trade_reviews VALUES
    (1, 'alice', '6f797a', TRUE),
    (2, 'alice', 'b523af', TRUE),
    (3, 'alice', 'fe1aaf', FALSE),
    (4, 'alice', 'f41bf3', TRUE),
    (2, 'bob', '0441ed', TRUE),
    (4, 'bob', '50f237', TRUE),
    (1, 'carol', 'ee52f5', FALSE),
    (3, 'carol', '4adb7c', TRUE);
```

这产生了准备处理的交易：

```noformat
   trade_id            approvals
          2                    2
          4                    2
```

## 参考资料

* 另请参阅[事件分组](../stream-processing/event-grouper.md)模式，了解`GROUP BY`操作的更一般讨论。
* 有关进一步讨论，请参阅[设计事件驱动系统](https://www.confluent.io/designing-event-driven-systems/)的第15章"构建流服务"。
