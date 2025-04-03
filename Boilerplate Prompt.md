
# Boilerplate Prompt

请帮我生成一个基于 .NET 8 的分层架构项目，项目名称为 `Boilerplate`，参考 ABP vNext 的模块化思想，结合六边形架构（Hexagonal Architecture）原则。要求将基础框架项目与具体业务项目分离，以提升复用性和解耦性。

## 架构与设计原则

### 模块化 & 六边形架构

- 每层通过接口定义“端口”，实现通过“适配器”注入。
- 基础设施依赖应用或领域的接口，**反转依赖方向**。
- 控制器不直接访问数据库，只通过 Application 层调用服务接口。

### 分层依赖约束

- 所有代码需分层合理，**不能跨层访问非接口定义的实现**。
- 所有交互应通过抽象接口 + 依赖注入进行。

### 接口定义建议

- 评估接口是否增加解耦或测试便利性，避免不必要的复杂度。
- 参考 DDD 和 ABP 的最佳实践，保持清晰建模语义。

## 技术要求

### CQRS 与事件驱动

- 所有命令/查询使用 MediatR 的 `IRequestHandler`。
- 所有领域事件处理使用 `INotificationHandler`。
- 命令与查询处理应分离为不同的类，符合 CQRS 架构风格。

### 数据库 ORM

- 使用 EF core 和 dapper 分别实现数据库访问示例项目的数据库访问，可以通过依赖注入进行切换。

## 项目划分与目录结构

### 项目分为通用基础类库和实例项目两部分

#### /framework - 通用基础类库（可复用）

该目录**不包含任何具体业务逻辑**，可以被多个业务项目引用：

- **Boilerplate.Domain.Shared**：领域层

  这个项目包含了一些常量、枚举和其他对象，它们实际上属于领域层的一部分，但由于需要被解决方案中的所有层/项目使用，因此被单独提取出来。该项目不依赖于解决方案中的其他项目，而其他所有项目都直接或间接依赖于它。
  
- **Boilerplate.Domain**：领域层

  包含实体基类（接口）、聚合根基类（接口）、值对象基类（接口）、领域服务基类（接口）、领域事件基类（接口）、仓储接口等。

  对于**实体, 聚合根, 值对象**等，如果需要跨项目复用或测试隔离，建议除了定义基类之外，同时定义对应的接口。
  依赖于 `.Domain.Shared` ，因为该项目中定义了常量、枚举和其他对象，当前项目需要使用它们。

- **Boilerplate.App.Contract**：应用层

  该项目主要包含应用层的应用服务接口和数据传输对象（DTO）。其目的是将应用层的接口与实现分离，从而可以将该接口项目作为契约包共享给客户端使用。
  
  该项目依赖于 `.Domain.Shared`，因为在应用服务接口和 DTO 中可能会使用该项目中定义的常量、枚举和其他共享对象。
  
- **Boilerplate.App**：应用层

  该项目包含了在 `.App.Contracts` 项目中定义的应用服务接口的具体实现。 

  该项目依赖于 `.Application.Contracts` ，以便实现其中定义的接口并使用其 DTO。

  同时也依赖于 `.Domain` ，以便在执行业务逻辑时使用领域对象（如实体、仓储接口等）。

- **Boilerplate.Infra**：通用基础设施实现  
  包含通用仓储基类、MediatR 分发器、CQRS 抽象（命令/查询/处理器）,工作单元接口实现等。

#### /samples/OrderService - 示例业务服务模块

该目录是示例业务模块，**引用 framework 中的抽象定义并完成具体实现**：

- **OrderService.Domain.Shared**: 共享部分
- **OrderService.App.Contract**：定义业务逻辑相关的接口、数据传输对象（DTOs）、枚举常量、事件、验证和特性等，以实现模块化和松耦合。  
- **OrderService.Domain**：订单聚合与领域逻辑  
- **OrderService.App**：订单用例与命令查询逻辑  
- **OrderService.Infra**：订单数据库与仓储实现  
- **OrderService.WebApi**：暴露 API 接口，配置启动项  

### 命名空间与目录建议

命名空间应与物理目录结构一致：

``` bash
src/framework
├── Boilerplate.Domain
├── Boilerplate.Domain.Shared
├── Boilerplate.App
├── Boilerplate.App.Contract
├── ...
└── Boilerplate.Infra

src/samples/OrderService
├── OrderService.Domain
├── OrderService.Domain.Shared
├── OrderService.App
├── OrderService.App.Contract
├── OrderService.Infra
├── ...
└── OrderService.WebApi

src
└── Boilerplate.sln
```

命名空间示例：

- 框架项目：`Boilerplate.Domain`, `Boilerplate.App`  
- Sample 项目：`OrderService.Domain`, `OrderService.App`  

## 示例模块内容 - OrderService
  
通过一个OrderService以复现一个完整的订单创建流程。

### 端到端示例要求

1. 接收 API 请求（POST `/api/orders`）。
2. 将请求转换为 `CreateOrderCommand`。
3. 应用服务接收命令，调用 `IOrderRepository`。
4. 保存订单聚合并触发 `OrderCreatedEvent`。
5. 使用 MediatR 派发并处理事件。

整个流程应覆盖 Controller → AppService → Domain → Infrastructure → EventHandler。

### 实例项目结构

#### OrderService.Domain

- `Order` 聚合根  
- `OrderItem` 子实体  
- `Address` 值对象（使用 `record`）  
- `OrderCreatedEvent` 领域事件  
- `IOrderRepository` 接口（定义于 framework）  

#### OrderService.Application

- 应用服务：`OrderAppService`  
- CQRS 命令：`CreateOrderCommand`  
- CQRS 查询：`GetOrderByIdQuery`  
- DTO：`OrderDto`, `CreateOrderDto`  
- 事件处理器：`OrderCreatedEventHandler`  

#### OrderService.Infrastructure

- `OrderRepository`：实现 `IOrderRepository`  
- `OrderServiceDbContext`：EF Core 上下文  
- `OrderEntityConfiguration`：Fluent 配置  
- 注册 MediatR 与依赖注入容器  

#### OrderService.WebApi

- `OrdersController`：提供 REST API  
- Swagger 配置  
- 程序入口和服务注册（Program.cs）

## 通用编码规范

### 面向对象原则（OOP）

请遵循经典面向对象设计原则，包括：

- 单一职责原则（SRP）
- 开闭原则（OCP）
- 里氏替换原则（LSP）
- 接口隔离原则（ISP）
- 依赖倒置原则（DIP）

### 注释要求

- 所有类和方法应包含 **XML 注释**，简明描述其职责与作用。

### C# 现代编码规范

- 使用 `record` 定义不可变 DTO 或值对象。
- 使用 `readonly` 字段代替可变字段。
- 使用 `init` 设置只读属性。
- 使用 `private set` 控制聚合内部状态变更。
