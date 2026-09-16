---
name: database-sql-validate
description: 校验MyBatis SQL脚本、DDL语句，检查风险SQL、语法问题，对齐项目数据库规范。
---

# 技能目标
检查SQL是否符合 project-rules.md 数据库规范，识别高危SQL与语法问题。

# 检查项清单
1. 禁止 SELECT *
2. UPDATE / DELETE 是否缺少 WHERE 条件
3. 字段、表名命名是否符合项目约定
4. 检查SQL注入风险（动态字符串拼接）
5. 检查缺少索引的大表全表扫描风险
6. 校验字段类型、约束与docs/legacy/db表字典保持一致

# 执行步骤
1. 接收待校验SQL文本（DDL / MyBatis xml内SQL）
2. 逐项执行上面的检查规则
3. 输出风险等级：高风险 / 警告 / 通过
4. 高风险项必须明确标注风险后果，给出修改方案

# 输出格式