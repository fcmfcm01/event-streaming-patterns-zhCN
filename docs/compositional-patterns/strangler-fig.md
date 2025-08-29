---
seo:
  title: 绞杀
  description: 通过逐步替换旧系统的功能来迁移单体应用程序。
---

# 绞杀

绞杀模式是一种用于将单体应用程序迁移到微服务架构的策略。它通过逐步替换旧系统的功能来实现这一点，而不是一次性重写整个应用程序。

## 问题

如何在不中断服务的情况下将单体应用程序迁移到微服务架构？

## 解决方案
![strangler-fig](../img/strangler-fig-a.svg)

绞杀模式通过以下步骤实现迁移：

1. **识别功能边界**：将单体应用程序分解为逻辑功能模块
2. **逐步替换**：一次替换一个功能模块，而不是整个应用程序
3. **并行运行**：在迁移期间，新旧系统并行运行
4. **流量路由**：使用路由机制将流量从旧系统逐步转移到新系统
5. **验证和监控**：确保新系统按预期工作，然后完全移除旧功能

## 实现

绞杀模式可以使用[事件流平台](../event-stream/event-streaming-platform.md)来实现，特别是通过[变更数据捕获](../event-source/database-write-through.md)来同步新旧系统之间的数据。

### 使用Kafka Connect进行数据同步

```json
{
  "name": "legacy-db-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:mysql://legacy-db:3306/monolith",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "legacy-",
    "poll.interval.ms": "5000"
  }
}
```

### 使用事件流进行功能路由

```java
// 路由逻辑示例
public class StranglerFigRouter {
    
    public void routeRequest(Request request) {
        if (isFeatureMigrated(request.getFeature())) {
            // 路由到新的微服务
            routeToNewService(request);
        } else {
            // 路由到旧系统
            routeToLegacySystem(request);
        }
    }
    
    private boolean isFeatureMigrated(String feature) {
        // 检查功能是否已迁移到新系统
        return migratedFeatures.contains(feature);
    }
}
```

## 注意事项

* **数据一致性**：确保新旧系统之间的数据同步和一致性
* **回滚策略**：准备回滚机制以应对新系统的问题
* **监控和观察**：密切监控迁移过程中的系统性能和错误
* **团队协调**：确保开发团队之间的良好协调和沟通
* **测试策略**：实施全面的测试策略，包括集成测试和端到端测试

## 参考资料

此模式由Martin Fowler在[绞杀模式](https://martinfowler.com/bliki/StranglerFigApplication.html)中首次描述。它基于绞杀者无花果植物的生长方式，这种植物逐渐包围并最终取代宿主树。
