
# Boilerplate Prompt

请帮我生成一个基于 .NET 8 的分层架构项目，项目名称为 `Boilerplate`，参考 ABP vNext 的模块化思想，结合六边形架构（Hexagonal Architecture）原则。要求将基础框架项目与具体业务项目分离，以提升复用性和解耦性。

## 目录结构与项目划分

### 命名空间与目录建议

命名空间应与物理目录结构一致：

``` bash
/framework
├── Boilerplate.Domain
├── Boilerplate.Application
├── Boilerplate.Infrastructure
├── Boilerplate.Contract
├── ...
└── Boilerplate.Shared

/samples/OrderService
├── OrderService.Domain
├── OrderService.Application
├── OrderService.Infrastructure
├── OrderService.Contract
├── ...
└── OrderService.WebApi
```

命名空间示例：

- 框架项目：`Boilerplate.Domain`, `Boilerplate.Application`  
- Sample 项目：`OrderService.Domain`, `OrderService.Application`  

### 1. /framework - 通用基础类库（可复用）

该目录包含所有领域抽象、应用抽象、通用基础设施实现、共享内核，**不包含任何具体业务逻辑**，可以被多个业务项目引用：

- **Boilerplate.Domain**：领域层  
  包含实体基类、聚合根、值对象接口、领域事件基类、仓储接口等。  
  > 接口定义建议：
  > - **IEntity, IAggregateRoot, IValueObject**：如果需要跨项目复用或测试隔离，建议定义。
  > - 评估接口是否增加解耦或测试便利性，避免不必要的复杂度。
  > - 参考 DDD 和 ABP 的最佳实践，保持清晰建模语义。

- **Boilerplate.Application**：应用层  
  包含应用服务接口、CQRS 抽象（命令/查询/处理器）、DTO 接口等。

- **Boilerplate.Infrastructure**：通用基础设施实现  
  包含通用仓储基类、MediatR 分发器、工作单元接口实现等。

- **Boilerplate.Shared**：共享内核  
  包含通用异常、标记接口、分页、排序枚举、规范接口等。

- **Boilerplate.Contract**：契约层  
  定义框架级别的接口、数据传输对象（DTOs）、枚举常量、事件、验证和特性等，以实现模块化和松耦合。

### 2. /samples/OrderService - 示例业务服务模块（可扩展）

该目录是示例业务模块，**引用 framework 中的抽象定义并完成具体实现**：

- **OrderService.Shared**: 共享部分
- **OrderService.Contract**：定义业务逻辑相关的接口、数据传输对象（DTOs）、枚举常量、事件、验证和特性等，以实现模块化和松耦合。  
- **OrderService.Domain**：订单聚合与领域逻辑  
- **OrderService.Application**：订单用例与命令查询逻辑  
- **OrderService.Infrastructure**：订单数据库与仓储实现  
- **OrderService.WebApi**：暴露 API 接口，配置启动项  

## 架构与设计原则

### 模块化 & 六边形架构

- 每层通过接口定义“端口”，实现通过“适配器”注入。
- 基础设施依赖应用或领域的接口，**反转依赖方向**。
- 控制器不直接访问数据库，只通过 Application 层调用服务接口。

### 分层依赖约束

- 所有代码需分层合理，**不能跨层访问非接口定义的实现**。
- 所有交互应通过抽象接口 + 依赖注入进行。

## 技术要求

### CQRS 与事件驱动

- 所有命令/查询使用 MediatR 的 `IRequestHandler`。
- 所有领域事件处理使用 `INotificationHandler`。
- 命令与查询处理应分离为不同的类，符合 CQRS 架构风格。

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
