网上看到很多 AI Agent 开发的教程，基本是用 python 等语言自己实现，这种 Agent 和 AI 工具中用文字描述的 Agent 有什么区别（如 .github/agents/xxx.agent.md）

一句话核心区别：

- **Python 手写 Agent（代码式 / 过程式 Agent）：你自己实现【Agent 运行时（思考 - 行动 - 观察循环、状态、调度、工具执行）】，LLM 只是其中一个组件**
- **`.github/agents/xxx.agent.md`（AGENTS.md 体系，声明式 Agent 定义）：只写【角色、规则、能力、边界、提示契约】，**运行循环、调度引擎由外部 AI 工具（Cursor、Copilot、Aider、Devin 等）提供 **，md 文件是给这个引擎读的配置 / 指令文件agentdotmd...

>
> 注意：`xxx.agent.md` 不是一个独立可运行程序，它**不能单独跑起来**；Python Agent 本身就是一个可独立运行的程序。

## 对比表

表格

| 维度 | Python 手写 Agent（LangGraph/LangChain/ 原生 SDK） | `.agent.md` / AGENTS.md |
| --- | --- | --- |
| 本质 | 完整 Agent 应用程序：**自己实现 Orchestrator（编排循环）** | **声明描述文件、契约文档**；编排引擎在外部工具里（Cursor/Copilot 等） |
| 你写什么 | 循环逻辑、状态管理、工具调用封装、重试、分支判断、记忆持久化、路由、多 Agent 调度；prompt 只是代码里的字符串 | 自然语言描述：身份、任务目标、允许工具、目录权限、编码规范、测试命令、禁止行为、输出格式；没有控制流代码 |
| 控制权 | 全可控：什么时候调用 LLM、什么时候重试、怎么存记忆、怎么并行、怎么中断都由代码决定 | 控制权交给宿主工具引擎；你只能通过文字约束模型行为，**无法自定义底层循环逻辑** |
| 运行依赖 | 自己启动进程，直接调用 LLM API，自主驱动循环 | 必须被支持 AGENTS.md 规范的 AI 编码工具加载解析；工具读取 md，把内容塞进 System Prompt 里驱动内置 Agent 循环 |
| 调试方式 | 断点、日志、单步看每一步 tool 调用、状态变量；可以改造执行流程 | 只能看模型输出、看它是否遵守 md 里写的规则；**无法介入底层调度逻辑** |
| 版本变更影响 | 改逻辑要改代码、改框架、改依赖，需要发布部署 | 改 Agent 能力只改 markdown，Git diff 友好，无需重新构建程序；适合项目内随代码一起维护、团队协作 |
| 适用场景 | 复杂业务 Agent、多智能体协作、自定义状态机、私有工具、产品化对外服务 | 代码库内给 AI 编码助手的行为规范、项目内专用小 Agent、统一约束 Copilot/Cursor 在仓库里怎么干活GitHub |

## 容易混淆的两个点

1. **AGENTS.md ≠ 只是普通 system prompt**
   它是一套社区规范（Linux 基金会下 Agentic AI Foundation），工具会约定性地去仓库根目录、`.github/agents/`下自动发现这类文件，分层继承（子目录 md 覆盖上层），不只是一段复制粘贴的提示词GitHub。很多还支持标记可用命令、读写白名单、风险操作确认规则。
2. **声明式 Agent 也有两种形态，别混在一起**
    - 纯 `.agent.md`：**自然语言声明**，宿主引擎完全用 LLM 来理解规则；灵活性高，但一致性不稳定，规则越复杂越容易 “失忆”
    - 框架里的声明式 Agent（比如 CrewAI、OpenAI Agents SDK 的声明式定义）：用代码结构（json / 对象）声明 name、instructions、tools，**循环依然是框架帮你写好**，介于两者之间，比纯 md 稳定，比手写循环简单

## 举个直观例子

- Python 手写 Agent：你写 `while not done: think -> call_tool -> observe`，自己写 `read_file()`、`run_test()` 函数，模型只负责输出下一步动作，你代码负责执行、判断终止条件。
- `.github/agents/coder.agent.md`：你写一段文字：

>
> 你是本仓库后端代码审查 Agent；只能读取 src 目录，禁止修改 config；发现 bug 先写单元测试再修复；修复后自动运行 `mvn test`；修改核心类前必须先询问我确认。
> 然后打开 Cursor，Cursor 内置的 Agent 引擎自动读到这个 md，把它作为系统指令，**Cursor 自己内部的循环去驱动思考和工具调用**，你没有写任何循环代码。

## 怎么选

- 如果你要做独立服务、复杂工作流、多 Agent 协作、定制状态机：Python / 框架手写 Agent
- 如果你只是想让 Cursor / Copilot / Aider 在当前代码仓库里，固定一套行为规范、角色：用 `.agent.md`，成本极低，跟着代码一起托管

如果你需要，我可以给你一份标准的 `.github/agents/coder.agent.md` 模板，以及一个极简 Python Agent 循环对照版本。

昨天 23:02

它们有专业名词称呼吗，平时我们怎么称呼这种各种不同形式的 agent

# 核心标准术语（行业通用）

>
> 对应你问的两类：`.agent.md` 那一类 / Python 手写循环那一类

1. **Declarative Agent｜声明式 Agent**（`.github/agents/*.agent.md`、AGENTS.md、Copilot 声明 Agent、Cursor 仓库 Agent）Microsoft ...
    - 同义叫法：**Prompt Agent / Instruction Agent**（微软 Azure Foundry 叫法）、Repo Agent（圈内口语：仓库 Agent）
    - 一句话：**只声明目标、规则、权限、角色，不写控制流；编排循环由宿主工具引擎提供**
2. **Imperative Agent｜命令式 Agent（也叫 Programmatic Agent / Custom Engine Agent）**（LangGraph、Python 手写 ReAct 循环、OpenAI Agents SDK 自研编排）Microsoft ...
    - 同义叫法：**Code-first Agent、自定义引擎 Agent、自研编排 Agent**
    - 一句话：**代码里写死 / 定义思考 - 行动循环、状态流转、分支、重试逻辑，自己掌控 Orchestrator 编排器**

>
> 论文里也会直接成对写：**Declarative Agent ↔ Imperative Agent**，是很标准的一对对照名词

## 日常口语怎么说（程序员圈子，写代码 AI 圈子）

### 聊 `.agent.md` 这类

- 口语：**仓库 Agent、项目内 Agent、md Agent、提示词 Agent、配置式 Agent**
- 场景：“我在仓库丢一个 `coder.agent.md`，给 Cursor 用的声明式 Agent”

>
> 注意：很多人会随口叫它 **system prompt**，但严格来说不一样：AGENTS.md 是一套带规范、自动发现、权限白名单机制的声明定义，不是单纯一段 prompt。

### 聊 Python/LangGraph 手写那一类

- 口语：**代码 Agent、自研 Agent、框架 Agent、可独立部署 Agent**
- 场景：“这个是 LangGraph 写的命令式 Agent，自己跑 ReAct 循环”

---

# 容易混在一起的其他 Agent 名词（避免踩坑）

这些是**另一套分类维度**，不是上面的声明 / 命令式，经常交叉：

1. **ReAct Agent**：是**Agent 循环范式**（Reason+Act），不是部署形态。声明式、命令式 Agent 都可以跑 ReAct。
2. **Multi-Agent / 多智能体**：是架构（多个 Agent 互相转交任务），和你是 md 配置还是 Python 代码无关。
3. **Assistant（OpenAI Assistants）**：托管式 Agent 运行时，介于两者之间 —— 引擎 OpenAI 提供，你用 API 配置工具、指令，**属于托管式 Prompt Agent**。
4. **Workflow Agent / 工作流 Agent**：偏固定分支的编排（比如 Dify、n8n），很多是半声明半命令。
5. **BDI Agent（信念 - 愿望 - 意图）**：是经典老派 AI 学术分类，现在 LLM 圈子很少用。

## 一张速记表

表格

| 正式术语 | 英文 | 你手上的例子 | 圈内口语 |
| --- | --- | --- | --- |
| 声明式 Agent | Declarative Agent | `.github/agents/xxx.agent.md`，Cursor/Aider/Copilot 仓库 Agent | 仓库 Agent、md Agent、Prompt Agent |
| 命令式 / 程序化 Agent | Imperative / Programmatic Agent | Python+LangGraph，自己写 ReAct 循环 | 代码 Agent、自研编排 Agent |
| 自定义引擎 Agent | Custom Engine Agent | 同上（微软文档专用词） | 独立 Agent、服务化 Agent |

## 一句标准话术，面试 / 交流直接用

>
> 一类是**声明式 Agent（Declarative Agent）**，通过 Markdown/YAML 描述角色和约束，由 IDE / 助手内置编排引擎驱动，典型就是 AGENTS.md 规范；另一类是**命令式 Agent（Imperative Agent）**，开发者用代码实现 Orchestrator 编排循环、状态机和工具调度，比如 LangGraph 开发的 Agent。

如果你想要，我可以给你一段简短的英文版本，适合看国外文档、Discord、GitHub 讨论。