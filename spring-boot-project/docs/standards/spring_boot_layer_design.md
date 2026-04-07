# Spring Boot 后台服务分层架构设计规范

## 1.架构设计原则

我们结合了“六边形”模型和 DDD 设计范式，将后台服务设计为分层架构。主要包含一下分层：

### Adaptor Layer

- 关注各类输入输出技术，比如 Restful API、RPC、访问数据库、访问各类中间件等。通过能力供应商（Capability Provider）模式，为更稳定的层提供具体输入输出技术的实现，从而实现关注点分离。
- 根据数据流向分成两大类：
    - 由外向内的适配器叫做 driving adapter。
    - 由内向外的适配器叫做 driven adapter。

### Application Layer

- 关注与细粒度的领域逻辑无关的、横切关注点的逻辑。主要包括：
    - 接受来自客户端的请求，调用和协调领域层的逻辑来解决问题。
    - 将领域层的处理结果封装为更简单的粗粒度对象，作为对外 API 的参数。
    - 负责处理事务、日志、权限等等横切关注点。
- 应用层本身并不包含领域逻辑，而是对领域层中的逻辑进行封装和编排。应用层的逻辑称为应用逻辑。
- 封装应用逻辑的类通常没有状态，只有方法，一般称为应用服务，可以用 XXXService 的形式来命名。

### Domain Layer

- 领域层封装领域数据和逻辑。我们将领域层独立出来，以保证与领域模型的一致，从而让领域层独立演化。

### Common Layer

- 应用基础框架、通用工具等

## 2.架构示例

假设，我们需要为 Org 实体设计 API，比如创建、查询等等。

### 包与类

- xxx.xxx.xxx
    - adaptor
        - driving
            - restful
                - org
                    - OrgController.java
        - driven
            - db
                - org
                    - OrgRepositoryJdbc.java
    - application
        - org
            - service
                - OrgService.java
            - dto
                - CreateOrgRequest.java
                - OrgResponse.java
                - UpdateOrgBasicRequest.java
    - domain
        - org
            - entity    
                - Org.java
            - domainservice
                - OrgValidator.java
            - factory
                - OrgBuilder.java
                - OrgBuilderFactory.java
            - repository
                - OrgRepository.java

### 类的设计规范

#### Adaptor 层

1. 控制器类 OrgController

- 注入对应的领域实体服务类，比如 `OrgService`
- 实现领域实体 API 的方法，比如 GET、POST、PUT、DELETE 等 API
- 在接口方法中，调用服务类的方法，实现对应的功能，比如 `orgService.addOrg(request, userId);`
- 在接口方法中，通常用 Application 层的 DTO 类来接受参数，比如 `OrgDto`

2. Repository 实现类 OrgRepositoryJdbc

- 通常需要实现对应的 Domain 层的 Repository 接口，比如 `OrgRepository`
- 需要调用具体的操作数据库的库的方法来完成具体的数据库操作，比如 `public Optional<Org> findByIdAndStatus(Long tenantId, Long id, OrgStatus status)` 等
- 命名上，通常采用在对应的接口的名称后面，追加能够表达具体实现手段的名词，比如 `OrgRepositoryJdbc` 、`OrgRepositoryRedis` 等 

#### Application 层

1. 实体服务类

- 通常需要注入所用到的各种 Repository 类
    - 比如 `UserRepository` 、`OrgRepository` 、`TenantRepository` 等，这取决于应用逻辑中需要哪些实体的 Repository
- 实现能够满足对应 Controller 的方法
    - 比如 `public OrgDto addOrg(OrgDto request, Long userId)`
- 通常需要实现 Entity 类和 DTO 类的转换方法
    - 比如 `private OrgDto buildOrgDto(Org org)` 、`private Org buildOrg(OrgDto request, Long useId) `
- 在实现具体功能的时候，要尽量保证 Service 这一层的轻量，将于领域逻辑相关的代码放在 Domain 层，仅仅在 Service 中调用即可
    - 比如，在实现一些于领域逻辑相关的规则的时候，就可以将代码放在 Domain 层的 `OrgValidator` 中

2. DTO 类

- 类的属性通常与对应的 Entity 类很类似
- 这种类很单薄，只是用于数据转换，没有任何领域逻辑代码
- 不同接口所需要的数据范围不同，为了保证封装性，我们针对不同的接口设计不同的 DTO 类
    - 比如 `CreateOrgRequest` 仅仅用于创建 Org 的接口，`UpdateOrgBasicRequest` 仅仅用于更新 Org 基本信息的接口
    - 比如，使用 `OrgResponse` 来承载需要返回 Org 的数据 

#### Domain 层

1. 领域对象类

- 所有属性都应该设置成为 `private`，从而满足领域实体类的数据封装性
- 只为那些可以修改的属性保留 setter，其他的只有 getter，成为只读属性。
- 领域对象类通常是状态的
- 领域对象类绝对不能直接操作数据库

2. 领域对象 Builder 类

- DDD 认为，领域对象的创建逻辑也是领域层的一部分。如果创建领域对象的逻辑比较简单，可以直接用对象的构造器来实现。但是如果比较复杂，就应该把创建逻辑放到一个专门的机制里，来保证领域对象的简洁和聚焦
- 我们利用工厂模式或者Builder模式来实现复杂逻辑的对象创建

3. 领域服务 Domain Service 类

- 复杂的领域逻辑都应该建立对应的、合适的领域服务类，而不是应该放在上层的 Service 中
    - 比如 `OrgValidator` 中可以承载关于 Org 创建、更新等操作的校验规则

4. Repository 接口

- 定义上层 Service 类能够调用操作 Entity 的方法
    - 比如 `Optional<Org> findByIdAndStatus(long tenantId, Long id, OrgStatus status);`
    - 比如 `Org save(Org org);`
    - 比如 `boolean existsBySuperiorAndName(Long tenant, Long superior, String name);`
