---
seo:
  title: 事件协作
  description: 事件协作允许服务和应用程序在事件流平台之上围绕单个业务工作流进行协作
---

# 事件协作

构建分布式业务工作流需要协调多个服务和[事件处理应用程序](../event-processing/event-processing-application.md)。随着复杂工作流通过各种状态转换，业务操作和反应必须异步协调。

## 问题

我们如何以允许组件独立演进的方式构建分布式工作流？

## 解决方案
![event-collaboration](../img/event-collaboration.svg)

事件协作允许服务和应用程序在[事件流平台](../event-stream/event-streaming-platform.md)之上围绕单个业务工作流进行协作。服务组件将[事件](../event/event.md)发布到[事件流](../event-stream/event-stream.md)，作为工作流中步骤完成的通知。事件还服务于携带工作流状态信息的额外目的，这些信息被下游组件在工作流的后续步骤中使用。该过程重复进行，直到工作流完成。

## 注意事项

* 在事件协作中，业务工作流进展编排的逻辑是分散的，分布在许多组件中。这与传统的编排设计形成对比，在传统编排设计中，逻辑被隔离在一个专门的"控制器"服务中，该服务协调工作流组件的操作和反应。

* 使用事件协作，一些工作流组件需要能够在生成自己的事件后的一段时间内确定工作流的状态。一个经典的例子是订单请求服务，它生成新的订单请求事件，并希望在订单完成时得到通知。这些事件需要通过分布式工作流进行关联，以支持此类功能。[关联标识符](../event/correlation-identifier.md)模式描述了一种在异步处理事件时通过遍历工作流内事件的全局标识符来耦合事件的方法。

## 参考资料

* [Martin Fowler的事件协作](https://martinfowler.com/eaaDev/EventCollaboration.html)
