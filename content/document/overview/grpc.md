---
date: 2020-02-01T11:00:00+08:00
title: gRPC描述符
menu:
  main:
    parent: "document-overview"
weight: 211
description : "gRPC描述符"
---

> 备注：内容摘录自 CloudState概述：https://cloudstate.io/docs/user/features/grpc.html

Cloudstate实体使用[gRPC](https://grpc.io/)描述符指定其接口。这是一个示例描述符：

```protobuf
syntax = "proto3";

import "google/protobuf/empty.proto";
import "cloudstate/entity_key.proto";

package example.shoppingcart;

service ShoppingCartService {
    rpc AddItem(AddLineItem) returns (google.protobuf.Empty);
    rpc RemoveItem(RemoveLineItem) returns (google.protobuf.Empty);
    rpc GetCart(GetShoppingCart) returns (Cart);
}

message AddLineItem {
    string user_id = 1 [(.cloudstate.entity_key) = true];
    string product_id = 2;
    string name = 3;
    int32 quantity = 4;
}

message RemoveLineItem {
    string user_id = 1 [(.cloudstate.entity_key) = true];
    string product_id = 2;
}

message GetShoppingCart {
    string user_id = 1 [(.cloudstate.entity_key) = true];
}

message LineItem {
    string product_id = 1;
    string name = 2;
    int32 quantity = 3;
}

message Cart {
    repeated LineItem items = 1;
}
```

该描述符提供购物车实体。它支持三种不同的命令`AddItem`，`RemoveItem`和`GetCart`。

### 指定实体键

在上面的描述符中要注意的最重要的事情是实体键注释。用作rpc命令输入的每条消息都有一个-这是Cloudstate的要求，所有入站命令消息都必须包含一个实体键。

Cloudstate使用实体键来了解命令所针对的实体实例。在上面的示例中，使用的实体键为`user_id`。这意味着`user_id`系统中每个实体都有一个购物车实体。当收到给定实体键的命令时，如果实体的状态尚未建立，Cloudstate将使用该实体类型的协议建立对 user function 的gRPC流式调用，并且针对实体键接收的任何命令都将通过该调用发送。

Cloudstate实体键必须是字符串。当将非字符串类型指定为实体键时，它将以代理特定的方式转换为字符串。因此，为了获得最大的可移植性，建议仅将字符串用作实体键。如果将多个字段指定为实体关键字，则将这些字段以代理特定的方式串联在一起。

### 转码HTTP

Cloudstate代理使用[此处](https://cloud.google.com/endpoints/docs/grpc/transcoding)描述的Google转码注释，支持将gRPC转码为HTTP/JSON 。使用此功能，您可以使用HTTP/ JSON使用实体的gRPC接口。