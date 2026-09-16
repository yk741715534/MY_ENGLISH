Copilot AI 规则体系说明
本文档用于说明本项目 GitHub Copilot Agent 全套规则文件架构，包含规则分工、AI 工作机制、目录约束、使用规范。
目的：统一团队 AI 开发标准、避免规则混乱、防止 AI 乱改文档、保证重构业务一致性。

---
## 1. 整体架构总览
   本项目采用 「1套全局规范 + 1个全局路由 + 2个专用Agent + 独立知识库」 的企业级 Copilot 架构：
- project-rules.md：项目永久编码规范（所有AI、所有人必须遵守）
- 根目录 AGENTS.md：全局兜底 + 任务路由分发
- backend-agent.agent.md：日常开发专用 Agent（新增功能、Bug 修复、迭代开发）
- java-refactor-agent.agent.md：旧系统重构专用 Agent（重写、迁移、业务对齐）
- docs/legacy：旧系统只读知识库（业务/表结构/旧代码/设计文档）
- docs/new：AI 重构产出目录（新方案、新设计、SQL 变更）

---
## 2. 核心文件分工（最重要、无重复）
   2.1 .github/instructions/project-rules.md 【静态规范】
   定位：项目「法律条文」
- 存放：技术栈、包分层、命名、SQL、异常、日志、测试、安全规范
- 所有 Agent 默认自动加载、永久生效
- 定义：代码最终必须长成什么样
- 所有人、所有 AI 任务统一遵循
  禁止写入：工作流程、步骤、AI 行为、输出格式（交给 Agent 文件）
  2.2 根目录 AGENTS.md 【全局路由 & 兜底】
  定位：AI 总调度中心
- 负责识别任务类型，自动分发到对应专用 Agent
- 定义全局只读目录、全局禁止项、通用协作规则
- 非编码类任务（文档整理、需求梳理、咨询、代码阅读）由本兜底 Agent 处理
  不写任何具体编码流程、不写规范、不重复 rules
  2.3 .github/agents/backend-agent.agent.md 【日常开发 Agent】
  定位：常规业务开发工程师
  适用场景：
- 新增接口、新增模块、业务迭代
- 普通 Bug 修复、代码优化、补全单元测试
- 日常业务功能开发
  特点：轻业务对齐、重规范落地、快速迭代
  2.4 .github/agents/java-refactor-agent.agent.md 【重构专用 Agent】
  定位：旧系统重构专家
  适用场景：
- 旧系统功能迁移、模块整体重构
- 需要大量参考 legacy 旧业务、旧表、旧逻辑
- 需要保证 业务100%等价 的迁移任务
  特点：强制研读旧文档、强制先设计后编码、强制业务对齐、严格自检

---
## 3. Copilot 自动加载机制（关键）
- project-rules.md：全局 always 生效，所有对话自动读取
- 根目录 AGENTS.md：默认 Agent 会话自动加载，负责路由
- .github/agents/ 下所有 .agent.md：Copilot 自动扫描发现，下拉框可手动切换
- 同一时间只会启用一个 Agent，不会叠加冲突

---
## 4. 目录读写权限规范（强约束）
   4.1 /docs/legacy 【只读知识库】
   AI、所有人禁止修改、新增、删除任何文件
   存放：旧系统业务说明、数据库 DDL、表字典、旧代码片段、旧设计文档
   作用：重构时作为唯一业务真值来源
   4.2 /docs/new 【AI 产出目录】
   AI 所有设计类产出必须放这里：
- 重构方案设计
- 数据库变更脚本
- 新接口结构说明
- 模块设计文档

---
## 5. 任务使用推荐策略（团队统一用法）
   场景1：旧系统重构 / 模块迁移
   手动选择：java-refactor-agent
   优先级：业务等价 > 数据一致 > 代码规范
   场景2：新增功能 / 迭代开发 / 修普通bug
   手动选择：backend-agent
   优先级：规范统一 > 可维护性 > 简洁度
   场景3：文档整理 / 需求梳理 / 咨询答疑
   使用默认全局 Agent（自动路由）

---
## 6. 禁止行为（AI & 开发者统一遵守）
- 禁止在 Agent 文件中重复写入编码规范（统一由 project-rules 管理）
- 禁止 AI 修改 /docs/legacy 任何内容
- 禁止 AI 跳过设计、跳过自检、跳过单元测试直接编码
- 禁止私自变更旧系统业务逻辑、默认值、边界规则
- 禁止随意升级核心依赖、修改全局架构

---
## 7. 整套体系一句话总结
- project-rules.md：管代码长什么样
- AGENTS.md：管任务分给谁
- backend-agent：管日常开发怎么写
- java-refactor-agent：管旧系统怎么安全重构
- docs/legacy：管旧业务真值来源

---
## 8. Agent Skills
存放路径：`.github/skills/`
- 每个Skill独立子目录，主文件固定命名为 `SKILL.md`
- Skill = 可复用原子工具，供Agent调用，单一职责原则
- 示例：
    - `maven-build-check`：执行mvn compile，检查编译错误
    - `database-sql-validate`：静态校验SQL脚本风险

---
## 9. MCP 配置
文件路径：`.github/mcp.json`
- MCP = Model Context Protocol，给Agent提供外部命令行工具调用能力
- 配置了 maven 和 git 两个本地工具
- 约束：仅允许只读/安全编译类命令；禁止高危发布、强制提交类操作
- 配合 `.github/skills/` 中的技能使用，Skill 声明需要调用哪个mcp工具

---
## 10. 自定义斜杠命令（Slash Command）
VS Code Copilot自动识别两类自定义斜杠命令，输入 `/` 唤起下拉菜单：
1. `.github/prompts/*.prompt.md`
   文件名即为斜杠命令，适合固定任务一键启动，可指定默认使用哪个Agent。
2. `.github/skills/*/SKILL.md`
   skill的name自动注册为斜杠命令；可通过`user-invocable`控制是否在下拉菜单暴露。

### 常用内置斜杠
- `/agent`：切换自定义Agent（backend-agent / java-refactor-agent）

### 示例
- `/refactor-module` 启动模块重构任务
- `/gen-unit-test` 为当前文件生成单元测试
- `/maven-build-check` 手动执行maven编译校验（如开启user-invocable）

-------------------------
沟通方式
默认中文回复；代码、命令、变量名、文件路径保持英文
结论先行，简洁直接，不先铺垫背景
不谁媚，不夸"这是个很好的问题"，不以"当然可以"开头
给真实判断一一方案有问题直接指出，发现更好做法主动说明

Git
不自动git commit或git push，除非我明确要求
提交前先展示将要提交的变更摘要
commit message使用简洁英文

红线操作
以下操作即使在 auto-accept 模式下也必须先问我：
删除文件、目录或git历史
修改.env、密钥、token、证书、Cl/CD配置
git push、git rebase、git reset --hard、强制推送
公开发布（npm pub1ish、生产部署等）
-------------------------

Prompt 编写技巧
1.任务描述要具体，不要模糊
2.引用已有代码作为参考
3.先让AI制定计划，确认后再执行
4.一次只做一件事

-------------------------
your-springboot-project/
├── .github/
│   ├── instructions/
│   │   └── project-rules.md
│   ├── agents/
│   │   ├── backend-agent.agent.md
│   │   └── java-refactor-agent.agent.md
│   ├── skills/                     # ✅ 新增：项目Skill目录（官方推荐）
│   │   ├── maven-build-check/      # 技能1：maven编译校验skill
│   │   │   └── SKILL.md
│   │   ├── database-sql-validate/   # 技能2：SQL校验skill
│   │   └── java-test-generator/    # 技能3：单元测试生成skill
│   ├── mcp.json
│   └── workflows/
├── AGENTS.md
├── docs/
│   ├── legacy/
│   └── new/
├── src/
├── pom.xml
└── README.md

-------------------------------
关于MCP
安全限制建议（非常重要）
在你的 skill 和 agent 规则里增加约束：
maven 只允许执行 compile、test；禁止 deploy、release、mvn clean package（可选放开）
git 只允许读操作：git status、git diff、git log；禁止 push、commit、reset、force 操作