---
seo:
  title: 数据库旁路写入
  description: 同时更新数据库中的值并在事件流平台中创建关联事件。
---

# 数据库旁路写入

直接写入数据库的应用程序可能希望为每个写入操作向[事件流平台](../event-stream/event-streaming-platform.md)产生关联的[事件](../event/event.md)，允许下游[事件处理应用程序](../event-processing/event-processing-application.md)被通知并消费[事件](../event/event.md)。

## 问题

我如何以最少的努力更新数据库中的值并创建关联事件？

## 解决方案

![database-write-aside](../img/database-write-aside.svg)

写入数据库，然后写入Kafka。在数据库事务的最后一步执行对Kafka的写入，以确保原子双重提交（如果对Kafka的写入失败则中止事务）。

## 实现

```
//启用事务
db.setAutoCommit(false);

try{
   //插入数据库
   sql = db.prepareStatement("insert into mydb.events values (?)");
   sql.setString(event.toString());
   sql.executeUpdate();

   //插入Kafka
   producer.send(event.key(), event.value());

   //提交到数据库
   db.commit();
} catch (SQLException e ) {
   db.rollback();
}
```

## 注意事项

在其默认形式中，此模式为大多数用例保证双重写入。但是，如果数据库事务在提交时失败（比如，因为数据库服务器崩溃），对Kafka的写入无法回滚，除非已启用事务。对于许多用例，这种可能性是可以容忍的，因为一旦故障修复，双重写入可以重试，而且大多数事件消费者无论如何都会实现幂等性。但是，应用程序程序员需要意识到没有严格的保证。

像Kafka这样的事务性消息系统可以用来提供更强的保证，只要所有事件消费者都启用了事务功能。

## 参考资料

* 有关将数据库变更写入事件流的替代示例，请参阅[数据库直写](database-write-through.md)
