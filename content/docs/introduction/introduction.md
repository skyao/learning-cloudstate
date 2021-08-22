---
title: "CloudState概述"
linkTitle: "概述"
weight: 101
date: 2021-01-18
description: >
  CloudState概述
---

![](images/logo-withtext.png)

### CloudState的背景

来自 https://cloudstate.io/ 首页的介绍：

> “We predict that serverless computing will grow to dominate the future of cloud computing.”
> 
> “我们预测无服务器计算将增长，从而统治云计算的未来。”
> 
> —Berkeley CS Dept, [Cloud computing simplified: a Berkeley view on serverless computing](https://arxiv.org/abs/1902.03383)

在Kubernetes生态系统中，将有状态服务，快速的数据/流和响应（reactive）技术的功能带入Cloud Native生态系统，以真正的弹性可扩展性，高可靠和全局部署，打破了阻碍 serverless 平台进行通用应用开发的最终障碍。

当今的 serverless 运动非常关注底层基础设施的自动化，但是在某种程度上它已经忽略了应用层上同样复杂的需求，在这些层上，向快速数据，流传输和事件驱动的有状态架构的迁移创造了各种生产中操作系统的新挑战。

无状态 function 是在云计算工具包中占有一席之地的出色工具，但是对于 serverless 而言，要想实现业界对 serverless 世界的要求，同时又使我们能够构建以数据为中心的实时应用，我们不能继续忽略分布式系统中最棘手的问题：管理状态-您的数据。

Cloudstate项目迎接了这一挑战，为 serverless 2.0铺平了道路。它由两部分组成：

- 标准工作：定义规范，用户 function 和后端之间的协议以及TCK。
- 参考实现：用不同的语言实现后端和一组客户端API类库。

Cloudstate的参考实现借力运行在 Kubernetes 上的 Knative，GRPC，Akka Cluster，以及GraalVM，允许应用不仅能高效的伸缩，也在在大规模场景下可靠地管理分布的状态，同时保持其数据一致性的全球或本地级别，对于整个开放新的可寻址用例范围。

个人总结：Serverless要实现大的愿景，就必须搞定有状态这个大难题。这就是CloudState项目的目标和价值。

### CloudState介绍

Cloudstate是一种规范，协议和参考实现，用于提供适用于 serverless 计算的分布式状态管理模式。当前支持和设想的模式包括：

- Event Sourcing
- 无冲突的复制数据类型（[CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)）
- Key-Value存储
- P2P消息传递
- CQRS读取侧投影

Cloudstate是多语言的，这意味着服务可以用任何支持gRPC的语言编写，并提供特定于语言的库，这些库允许每种语言的惯用模式。Cloudstate可以单独使用，也可以与Service Mesh结合使用，或者可以预见它将与其他 serverless 技术，例如[Knative](https://knative.dev/)集成在一起。

