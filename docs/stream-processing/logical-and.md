---
seo:
  title: 逻辑与
  description: 两个或多个事件流的逻辑与使用流连接合成新事件。
---

# 逻辑与

[事件流](../event-stream/event-stream.md)在考虑在一起时变得更有趣。通常，当两个单独的[事件](../event/event.md)发生时，会触发我们想要捕获的新事实。只有当有订单*和*成功支付时，产品才能被派送。如果某人下注*并且*他们的马赢了，那么我们就向他们转账。

我们如何组合来自几个不同事件流的信息并用它来创建新事件？

## 问题

当两个或更多相关事件到达不同事件流时，应用程序如何触发处理？

## 解决方案
![logical AND](../img/logical-and.svg)

多个事件流可以连接在一起，类似于关系数据库中的连接。我们监视事件流并记住它们最近的事件（例如，通过内存缓存或本地或网络存储设备）一段时间。每当新事件到达时，我们将其与其他最近捕获的事件一起考虑，并寻找匹配项。如果我们找到一个，我们就发出一个新事件。

对于流-流连接，重要的是要考虑我们认为什么是"最近"的事件。我们不能将全新的事件与任意旧的事件连接；连接潜在无限流将需要潜在无限内存。相反，我们决定一个保留期，该保留期算作"足够新"，并且只在该期间内保留事件。这通常很好——例如，支付通常在下订单后很快发生。如果在一小时内没有通过，我们可以合理地期望不同的流程来追查用户的更新支付详细信息。

## 实现

作为示例，想象一家银行捕获其网站的`logins`和ATM的`withdrawals`。欺诈部门可能很想听到同一个`user_id`在一个国家登录网站，并在同一天在另一个国家进行提款。（这不一定是欺诈，但肯定可疑！）

要实现此示例，我们将使用[Apache Flink® SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/sql/gettingstarted/)。我们从两个事件流开始：

```sql
-- 为了简单起见，我们假设IP地址已经转换为国家代码。

CREATE TABLE logins (
  user_id BIGINT NOT NULL,
  country_code STRING
);

CREATE TABLE withdrawals (
  user_id BIGINT NOT NULL,
  country_code VARCHAR,
  amount DECIMAL(10,2),
  success BOOLEAN
);
```

我们现在可以连接这两个事件流。具有相同`user_id`的事件被认为是同一个人的登录：

```sql
CREATE STREAM possible_frauds AS
    SELECT l.user_id, l.country_code, w.country_code, w.amount, w.success
    FROM logins l
    JOIN withdrawals w
    ON l.user_id = w.user_id
    WHERE l.country_code <> w.country_code;
```

在一个终端中查询该表：

```sql
SELECT * FROM possible_frauds;
```

在另一个终端中向`logins`和`withdrawals`表插入一些数据：

```sql
INSERT INTO logins VALUES
    (1, 'gb'),
    (2, 'us'),
    (3, 'be'),
    (2, 'us');

INSERT INTO withdrawals VALUES
    (1, 'gb', 10.00, true),
    (1, 'au', 250.00, true),
    (2, 'us', 50.00, true),
    (3, 'be', 20.00, true),
    (2, 'fr', 20.00, true);
```

这产生了一个需要进一步调查的可能欺诈案例流：

```
    user_id     country_code    country_code0       amount success
          1               gb               au       250.00    TRUE
          2               us               fr        20.00    TRUE
          2               us               fr        20.00    TRUE
```

## 注意事项

连接事件流相当简单。重要的考虑是我们需要多长的保留期，以及我们的连接将使用的资源量。规划这种权衡需要仔细考虑我们正在解决的具体问题。

对于长保留期，考虑将事件流连接到[投影表](../table/projection-table.md)。

## 参考资料

* 另请参阅[管道](../compositional-patterns/pipeline.md)模式，用于按顺序（而不是并行）考虑事件。
* 另请参阅[投影表](../table/projection-table.md)模式，这是在潜在无限时间段内考虑事件流的内存高效方式。
* 有关进一步讨论，请参阅[设计事件驱动系统](https://www.confluent.io/designing-event-driven-systems/)的第14章。
