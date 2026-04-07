# 测试规范

## 后台服务测试

### 单元测试

在 DDD (领域驱动设计) 范式下，单元测试不仅是质量保证工具，更是领域逻辑的文档。由于 DDD 强调业务逻辑高度集聚在领域层，单元测试的重点应从“测试代码”转向“验证业务契约”。

#### 单元测试的分层侧重点

在 DDD 架构中，并不是每一层都需要同等强度的单元测试：

- Domain (领域层)
    - 测试重点：核心业务规则、聚合根状态变迁、值对象相等性。
    - 核心技术：纯 JUnit 5 (无 Spring 依赖)。
    - 建议覆盖率：90% 以上。
- Application (应用层)
    - 测试重点：业务流程编排、权限检查、事务边界。
    - 核心技术：Mockito (Mock 掉 Repository)
    - 建议覆盖率：70% 左右
- Adaptor (适配器层)
    - 测试重点：适配器逻辑、映射逻辑。
    - 核心技术：通常通过集成测试覆盖
    - 建议覆盖率：较低

#### 领域层 (Domain) 测试规范 (重中之重)

A. 遵循“无框架”原则
- 规范：领域层的单元测试严禁启动 Spring Context（不要使用 @SpringBootTest）。
- 原因：领域逻辑应该是纯粹的 Java 对象。依赖 Spring 会导致测试运行极慢，破坏领域模型的纯净性。
- 做法：直接 new 领域对象，或者使用简单的 Mockito。

B. 针对“充血模型”的测试
- 规范：测试应验证行为而非状态。
- 错误示范：assertEquals(status, user.getStatus()) (仅验证 setter/getter)。
- 正确示范：user.lock(); assertTrue(user.isLocked()); (验证业务动作 lock)。

C. 值对象 (Value Object) 的不可变性
- 规范：必须测试值对象的相等性（Equals/HashCode）和不可变性。
- 案例：验证修改 Address 对象会返回一个新实例，而不是修改原实例。

#### 应用层 (Application) 测试规范

A. 依赖模拟 (Mocking)
- 规范：使用 Mockito 模拟所有的 Repository 和外部服务接口。
- 重点：验证 Application Service 是否正确调用了领域对象的方法，以及是否正确触发了领域事件。

B. 异常处理测试
- 规范：必须覆盖业务异常路径。
- 案例：当 Repository 返回空时，应用层是否正确抛出了 EntityNotFoundException。

#### 测试命名与结构 (Given-When-Then)

为了让测试代码成为“活的文档”，建议采用语义化命名：

- 命名模版：should_[期望结果]_when_[某种情境]
    - 例如：should_throw_exception_when_order_amount_is_negative()
- 结构规范：
- Given: 准备领域对象、聚合根及 Mock 行为。
- When: 执行被测的领域行为。
- Then: 断言状态改变、返回值或领域事件的发布。


### 集成测试

我们推行以下两种集成测试模式：

#### 持久层集成测试 (Repository Test)

- 目标：验证 JPA 映射、SQL 查询、聚合根持久化是否正确。
- 工具：@DataJpaTest + Testcontainers。
- 优点：不启动整个 Web 环境，只启动数据库相关的 Bean，速度快。

#### 全链路集成测试 (End-to-End Test)

- 目标：验证从 API 入口到数据库落地的完整业务流程。
- 工具：@SpringBootTest(webEnvironment = RANDOM_PORT) + Testcontainers。