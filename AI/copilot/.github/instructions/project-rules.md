# 项目全局编码与架构规范
> 本文件为项目永久规范，所有代码新增、修改、重构、Bug修复、单元测试均必须遵守。
> 参考资料位置：`docs/legacy/` 目录为旧系统业务、表结构、旧代码参考，**只读，禁止修改**。
> 所有业务逻辑、字段含义、原有约束以 docs/legacy 内文档为准；代码实现遵守本规范。

## 1 技术栈约定
- JDK：17
- 框架：SpringBoot 3.x
- 持久层：MyBatis / MyBatis-Plus
- 构建工具：Maven（pom.xml）
- 测试：JUnit 5 + AssertJ + Mockito
- 工具库：Lombok（禁止手写get/set/toString）
- 校验：Jakarta Validation（入参注解校验）
- 日志：SLF4J + Logback，禁止直接使用 System.out / System.err

> 除非人工明确审批，**禁止随意升级pom.xml核心依赖版本**。

## 2 包分层规范（严格遵循）
com.company.project
├── config // Spring 配置类、Bean、拦截器、全局异常、跨域配置
├── controller // HTTP 接口层，只做入参校验、请求转发、返回封装，不写业务逻辑
├── service // 业务接口定义
│ └── impl // 业务接口实现，核心业务逻辑在此
├── repository // Mapper 接口，数据库访问层
├── entity // 数据库实体，与数据库表一一映射
├── dto // 入参 DTO（Controller 接收前端请求）
├── vo // 出参 VO（返回给前端的数据模型）
├── mapper // MyBatis XML 文件（resources 下）
├── exception // 自定义业务异常、全局异常处理器
├── common // 通用常量、工具类、枚举、统一返回结果
└── util // 纯工具方法，无业务状态，无数据库依赖


### 分层约束
1. Controller → Service → Repository，**禁止跨层直接调用**：Controller不能直接调用Mapper；Mapper不能直接返回给前端。
2. Entity 仅用于数据库映射，**禁止直接返回Entity给前端**，必须转为VO。
3. DTO只用于接收前端入参，不传递到Repository层。
4. 业务枚举统一放在 `common/enums`，禁止魔法数字、魔法字符串硬编码。

## 3 命名规范
1. 类名：大驼峰 UpperCamelCase，名词，如 UserOrderService
2. 方法名：小驼峰 lowerCamelCase，动词+名词，如 getUserOrderById
3. 变量：小驼峰；常量：全大写，下划线分隔 `ORDER_STATUS_SUCCESS`
4. 包名：全小写，不使用下划线
5. Mapper XML文件：与Mapper接口同名，放在 `resources/mapper/`
6. 数据库表/字段：下划线命名，和entity映射保持一致

## 4 数据库与SQL规范
1. 禁止 `SELECT *`，明确列出需要查询的字段。
2. 所有SQL写在MyBatis XML，禁止在Java代码中拼接SQL字符串（防注入）。
3. 表、字段注释必须完整；业务含义参考 `docs/legacy/db/` 表字典。
4. 更新、删除操作必须带上条件，禁止无where的全表更新/删除。
5. 大事务谨慎使用，事务粒度尽量小；长事务禁止。
6. 索引变更、表结构变更，需要记录在 `docs/new/数据库变更记录.md`。

## 5 异常处理规范
1. 使用**全局异常处理器**统一捕获，Controller、Service内部尽量不要零散try-catch。
2. 业务场景抛出自定义 `BusinessException`，携带错误码+错误信息。
3. 底层异常日志必须打印完整堆栈，对外返回友好提示，禁止直接把原始异常堆栈返回前端。
4. 不要用异常做业务流程控制。

## 6 入参校验
1. Controller DTO使用Jakarta Validation注解校验（@NotBlank、@NotNull、@Size等）。
2. 复杂业务校验逻辑，放在Service层，不要写在Controller。
3. 接口参数校验失败，返回统一错误格式。

## 7 日志规范
1. 日志级别合理区分：ERROR用于异常，WARN用于业务告警，INFO用于关键业务流程，DEBUG用于调试。
2. 日志打印带上业务标识（订单号、用户ID等），方便排查。
3. 禁止日志打印明文敏感信息（手机号、身份证、密码）。
4. 生产环境默认关闭DEBUG日志。

## 8 单元测试规范
1. Service、Controller新增/修改代码，**必须配套JUnit5单元测试**。
2. 单元测试放置在 `src/test/java`，包路径和主代码一一对应。
3. Service测试优先Mock Mapper；Controller测试使用MockMvc。
4. 测试覆盖正常流程、边界条件、异常分支。
5. 测试代码同样遵守本项目命名规范。

## 9 代码风格 & 注释
1. 使用项目统一Java格式化配置，不要随意调整代码排版。
2. 类、公共方法写JavaDoc注释，说明功能、入参、返回值、异常场景。
3. 复杂业务逻辑行内增加单行注释，简单逻辑无需多余注释。
4. 禁止提交注释掉的废弃代码（直接删除，版本历史可回溯）。
5. 尽量减少长方法，单个方法控制在50行以内，过长则拆分子方法。

## 10 安全规范
1. 接口入参做防XSS、参数长度校验。
2. 禁止在代码硬编码密钥、数据库密码、token密钥；配置放yml环境配置。
3. 接口权限校验统一在拦截器/AOP，不要每个Controller手写权限判断。

## 11 重构专项规则（本次旧系统重构专用）
> 【重点，适配你当前场景】
1. 业务逻辑必须对齐 `docs/legacy/` 旧系统文档，**业务行为不能擅自变更**；如需修改原有业务规则，必须人工确认。
2. 重构优先保证**业务行为等价**，其次才是代码整洁、分层规范。
3. 遇到旧系统模糊、冲突的业务规则，**不要自行猜测**，先列出疑问，等待人工确认，再继续编码。
4. 新增设计方案、数据库变更、接口清单，输出到 `docs/new/`，不要写入legacy目录。
5. 重构完成后，同步补充对应的单元测试，验证和旧系统行为一致性。

## 12 禁止行为清单
- ❌ 禁止修改 `docs/legacy/` 下任何参考文档
- ❌ 禁止直接返回Entity实体给前端
- ❌ 禁止硬编码密钥、敏感配置
- ❌ 禁止无where条件的update/delete
- ❌ 禁止随意升级核心依赖版本
- ❌ 禁止在Controller写大量业务逻辑
- ❌ 禁止提交大量注释掉的废弃代码
- ❌ 禁止一次性大批量修改无关文件，尽量模块拆分、小范围提交

## 13 输出约定
每次完成任务，输出：
1. 本次变更文件清单
2. 关键业务风险点（需要人工复核的部分）
3. 编译/单元测试执行结果
4. 如涉及数据库变更，输出SQL变更脚本

