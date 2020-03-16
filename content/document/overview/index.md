---
date: 2020-02-01T11:00:00+08:00
title: CloudState概述
weight: 210
description : "CloudState概述"
---

> 备注：内容摘录自 CloudState概述：https://cloudstate.io/docs/user/features/index.html

## 高级概念

### 状态管理倒置

在传统的n层体系结构中，一层（应用层）将调用另一层（数据库层）来检索和操纵其状态。这种架构方法面临一些挑战，这些挑战与 serverless 的理念息息相关：

- 应用必须知道数据存储的位置，正在使用的技术，如何与之通讯等。
- 该应用负责处理与管理状态相关的错误，包括基础设施级别的故障以及领域级别的错误，例如并发更新和事务。

Cloudstate会反转此模型。应用代码不会调用状态管理系统，状态管理系统调用应用代码。如何访问数据，使用何种技术等已 100％ 成为状态管理系统的领域。数据访问错误也成为状态管理系统的领域-用户代码从此不必关注。事务问题，例如管理并发更新，缓存，分片，路由等，都成为状态管理系统而不是用户代码的关注。

在将状态管理问题从用户代码范围转移到基础设施范围时，我们可以简化用户需要编写的代码，从而实现更多自动化，并实现更多的深度监控。

换句话说，在反转状态管理时，应用代码不与数据库对话，更正确的说法是是数据库（或更确切地说，状态管理系统）与应用代码对话。应用代码不与数据库建立连接或任何调用，状态管理系统根据需要连接到应用代码。应用代码不会针对所需的状态发出查询，状态管理系统根据需要将状态传递给应用代码，作为需要处理的命令。

## 专业术语

下图说明了Cloudstate系统的不同组件如何组合在一起。

![该图显示了不同的Cloudstate概念如何融合在一起](images/overview.svg)

### 有状态服务

有状态服务是可部署的单元。它在Kubernetes中表示为 `StatefulService` 资源。它包含一个[User函数](https://cloudstate.io/docs/user/features/index.html#user-function)，并且可以引用有[状态存储](https://cloudstate.io/docs/user/features/index.html#stateful-store)。当Cloudstate Operator 处理它时，它将使用关联的 Kubernetes 服务将其转换为 Kubernetes 部署资源以进行访问。部署将注入Cloudstate [Proxy](https://cloudstate.io/docs/user/features/index.html#proxy)。

### 状态存储

状态存储是对数据存储（通常是数据库）的抽象。它在Kubernetes中表示为 `StatefulStore` 资源。多个[有状态服务](https://cloudstate.io/docs/user/features/index.html#stateful-service)可以使用单个存储来存储其数据，但是有状态服务可能不一定配置存储：如果有状态服务不具有任何持久化的状态，则它不需要有状态存储。

### User Function

User Function是用户编写，打包为Docker映像并部署为 [有状态服务](https://cloudstate.io/docs/user/features/index.html#stateful-service) 的代码。User function 公开了gRPC接口，该接口使用 Cloudstate [协议](https://cloudstate.io/docs/user/features/index.html#protocol)。注入的 Cloudstate [代理](https://cloudstate.io/docs/user/features/index.html#proxy) 使用此协议与用户功能对话。尽管用户功能确实实现了该协议，但是最终用户开发人员自己通常不会提供此实现，而是使用特定于 User function 所使用的语言的Cloudstate [支持库](https://cloudstate.io/docs/user/features/index.html#support-library)来实现协议，并提供一种语言特定的惯用API，供开发人员进行编码。

### 协议

Cloudstate协议是协议的开放规范，用于Cloudstate状态管理代理与 Cloudstate user function 通讯。Cloudstate项目本身提供了此协议的[参考实现](https://cloudstate.io/docs/user/features/index.html#reference-implementation)。该协议是使用gRPC构建和指定的，并支持多种不同的[Entity类型](https://cloudstate.io/docs/user/features/index.html#entity-type)。Cloudstate提供了一个TCK，可用于验证可用的[代理](https://cloudstate.io/docs/user/features/index.html#proxy)和[支持库的](https://cloudstate.io/docs/user/features/index.html#support-library)任何排列。

### 代理

Cloudstate代理作为 sidecar 注入到每个[有状态服务](https://cloudstate.io/docs/user/features/index.html#stateful-service)的 deployment 中。它负责状态管理，并暴露出由 [user function](https://cloudstate.io/docs/user/features/index.html#user-function) 实现的 [Entity Service](https://cloudstate.io/docs/user/features/index.html#entity-service)，给系统的其余部分提供 GRPC和REST服务，转换传入的调用为  [命令](https://cloudstate.io/docs/user/features/index.html#command) ，并发送到使用Cloudstate [协议](https://cloudstate.io/docs/user/features/index.html#protocol) 的 User function。代理通常将与同一有状态服务中的其他节点形成集群，从而允许高级状态管理功能，例如在单个有状态服务的多个节点之间的分片，复制和寻址通信。

### 参考实施

Cloudstate参考实现可实现Cloudstate [协议](https://cloudstate.io/docs/user/features/index.html#protocol)。它是使用 [Akka](https://akka.io/) 实现的，利用 Akka 的集群功能来提供Cloudstate有状态功能的可扩展和弹性实现。

### 支持库

尽管可以通过实现 Cloudstate [协议](https://cloudstate.io/docs/user/features/index.html#protocol) 中的gRPC接口来简单地实现 [user function](https://cloudstate.io/docs/user/features/index.html#user-function)，但该协议有些底层，并且并不特别适合于表达业务逻辑，而这些业务逻辑通常驻留在user function中。相反，鼓励开发人员为他们选择的语言（如果有）使用Cloudstate支持库。

### 命令

命令是 user function 收到的消息。命令可能来自[有状态服务之外](https://cloudstate.io/docs/user/features/index.html#stateful-service)，也可能来自其他有状态服务，其他非Cloudstate服务或外界，或者它们可能来自服务内部（作为调用或转发来自另一命令的命令处理的副作用）。

### 实体

一个[user function](https://cloudstate.io/docs/user/features/index.html#user-function)实现一个或多个实体。实体在概念上等效于类或状态类型。一个实体将具有可以处理命令的多个[实例](https://cloudstate.io/docs/user/features/index.html#entity-instance)。例如，user function 可以实现包含与聊天室关联逻辑的聊天室实体，并且特定的聊天室可以是该实体的实例，其中包含当前在该房间中的用户列表和发送给它的消息历史记录。每个实体都有一个特定的[Entity类型](https://cloudstate.io/docs/user/features/index.html#entity-type)，它定义了实体状态如何持久，共享以及其功能是什么。

#### 实体实例

[实体](https://cloudstate.io/docs/user/features/index.html#entity)的实例。实体实例由 [Entity Key](https://cloudstate.io/docs/user/features/index.html#entity-key) 标识，该键对于给定实体是唯一的。实体在[User function中](https://cloudstate.io/docs/user/features/index.html#user-function)保存状态，并且根据[实体类型](https://cloudstate.io/docs/user/features/index.html#entity-type) 保存此状态在gRPC流的上下文中。收到特定实体实例的命令后，[代理服务器](https://cloudstate.io/docs/user/features/index.html#proxy)将对该实体实例向[User函数](https://cloudstate.io/docs/user/features/index.html#user-function)进行新的流式gRPC调用。该实体实例收到的所有后续命令将通过该流调用发送。

#### 实体服务

实体服务是一种gRPC服务，它允许与[Entity](https://cloudstate.io/docs/user/features/index.html#entity)进行交互。该[代理](https://cloudstate.io/docs/user/features/index.html#proxy)使这项服务可用于其他Kubernetes服务和ingreses使用，而[用户功能](https://cloudstate.io/docs/user/features/index.html#user-function)提供了它的实现。请注意，该服务并非像普通的gRPC服务那样由 user function 直接实现，而是通过Cloudstate [协议](https://cloudstate.io/docs/user/features/index.html#protocol)实现的，该[协议](https://cloudstate.io/docs/user/features/index.html#protocol)通过状态管理功能（例如接收和更新状态的功能）丰富了传入和传出的gRPC消息。

#### 实体类型

[实体](https://cloudstate.io/docs/user/features/index.html#entity)使用的状态管理的类型。可用的类型包括[事件源](https://cloudstate.io/docs/user/features/index.html#event-sourced)和无[冲突的复制数据类型](https://cloudstate.io/docs/user/features/index.html#conflict-free-replicated-data-type)。每种类型都有自己的子协议，作为Cloudstate [协议的](https://cloudstate.io/docs/user/features/index.html#protocol)一部分，用于状态管理，以传达状态和特定于该类型的更新。

#### 实体键

用于标识[Entity](https://cloudstate.io/docs/user/features/index.html#entity)实例的键。所有[命令都](https://cloudstate.io/docs/user/features/index.html#command)必须包含实体键，以便可以将命令路由到该命令所来自的实体的正确实例。[实体服务](https://cloudstate.io/docs/user/features/index.html#entity-service)的gRPC描述符注释了[实体](https://cloudstate.io/docs/user/features/index.html#entity-service)的传入消息类型，以指示哪些字段包含实体密钥。

### Event Sourced

一种[实体](https://cloudstate.io/docs/user/features/index.html#entity)，使用事件日志存储其状态，并通过重播该日志来恢复其状态。在[Event Sourcing](https://cloudstate.io/docs/user/features/eventsourced.html)中将详细讨论这些内容。

### 无冲突的复制数据类型

一种[实体](https://cloudstate.io/docs/user/features/index.html#entity)类型，它使用无冲突的复制数据类型（CRDT）存储其状态，该类型在服务的不同节点之间复制。


