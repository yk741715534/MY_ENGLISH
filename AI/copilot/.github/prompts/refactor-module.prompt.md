---
description: 启动Java模块重构，自动加载java-refactor-agent，读取legacy文档，执行等价迁移
agent: JavaBackendRefactorAgent
---
请使用java-refactor-agent，对当前选中模块进行旧系统迁移重构。
严格遵循项目project-rules.md，优先保证业务等价。
参考docs/legacy内旧业务文档，先输出设计方案，再编码，最后调用skill做编译和SQL校验。