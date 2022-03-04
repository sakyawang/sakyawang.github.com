---
title: 微服务的设计模式
date: 2022-01-07 15:49:11
description: 微服务开发相关的设计模式
categories:
- 微服务

tags:
- 微服务
- 设计模式

cover: https://images.pexels.com/photos/4913367/pexels-photo-4913367.jpeg
---

微服务的目标是将应用程序分解为可独立部署的小自治服务，以提高应用程序发布的速度。 微服务体系结构也带来了一些挑战。 此处显示的设计模式可以帮助缓解这些难题。

![微服务设计模式](./Design-patterns-for-microservices/microservices-patterns.png)

[大使模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/ambassador) 可用于卸载常见的客户端连接任务，例如监视、日志记录、路由 (安全任务，例如，) 与语言无关的 TLS 连接。 代表服务通常部署为 sidecar (请参阅下面的) 。

[防腐层模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer) 实现新应用程序与旧应用程序之间的外观，以确保新应用程序的设计不受限于旧系统的依赖关系。

[前端专属的后端模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends) 不同类型的客户端（如桌面和移动）创建单独的后端服务。 这样一来，单个后端服务就不需要处理各种客户端类型的冲突要求。 此模式可以通过区分特定于客户端的问题来帮助简化每个微服务。

[隔板模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/bulkhead) 隔离每个工作负荷或服务的关键资源，例如连接池、内存和 CPU。 通过使用隔舱，单个 (或) 无法消耗所有资源，使其他工作负荷耗尽。 此模式通过防止一个服务导致的级联故障，提高系统的复原能力。

[网关聚合模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation) 将多个单个微服务的请求聚合为单个请求，减少使用者和服务之间的通信。

[网关卸载模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation) 使每个微服务能够将共享服务功能（例如 SSL 证书的使用）卸载到 API 网关。

[网关路由模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/gateway-routing) 路由使用单个终结点将请求路由到多个微服务，因此使用者无需管理多个单独的终结点。

[Sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) 将应用程序的帮助程序组件部署为单独的容器或进程，以提供隔离和封装。

[Strangler Fig](https://docs.microsoft.com/en-us/azure/architecture/patterns/strangler-fig) 通过逐渐将特定功能部分替换为新服务，支持应用程序的增量重构。

有关云设计模式的完整目录，Azure 体系结构中心 [云设计模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/)。
