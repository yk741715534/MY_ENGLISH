回到最初的问题，我一开始说声明式 Agent，它理解不是这类.github/agents/xxx.agent.md，难道是我表述有什么问题或是不清楚的地方吗

先把核心结论先说清楚：

>
> 不是你错了，也不是你表述严重有问题；**歧义点在于：「声明式 Agent」在现在有两套完全不同的主流语境**，我之前直接落到了 GitHub Copilot Custom Agent（`.github/agents/*.agent.md`）那套 DSL，是我先做了一个强假设，把你的「声明式 Agent」锁到了 Copilot 的载体里，导致跑偏。

分开讲两个语境，你就能看清差异在哪：

## 语境 A：GitHub Copilot Custom Agent（我之前默认的）

- 载体：`.github/agents/xxx.agent.md`
- 形式：YAML frontmatter（name、description、tools、permissionMode）+ 系统提示词
- 它**确实是一种声明式定义**：声明它是谁、有什么工具、权限是什么、该怎么思考
- 但是：它本质是 **提示词封装 + Copilot 平台的轻量 Agent 包装**，执行模型是平台黑盒，你只是声明它的行为，没法声明输入输出 Schema、流转规则、校验规则作为一等公民。

>
> 这是我脑子里第一跳过去的东西，所以后面顺着这个载体，把 Grilling、资产契约、NextSuggestion 全部塞到这个 md 文件里。

## 语境 B：你最开始想说的「声明式 Agent」（大概率是这个）

在 Agent / AI Agent 框架圈里说 **declarative agent（声明式 Agent）**，含义通常是另一回事：

>
> 不去写一大段系统提示词、不去写命令式的步骤（“你先做 A，再做 B，然后判断，如果缺东西就问用户…”）；
> 而是**声明它的契约：输入是什么 Schema、输出是什么 Schema、前置校验规则、澄清策略、依赖关系、后置动作**。
> Agent runtime（运行时引擎）去读这份声明，自动驱动流程：自动校验输入、自动触发 Grilling、自动调用 Agent、自动解析 asset、自动展示 NextSuggestion。

重点区别一句话：

- **命令式提示词**：告诉大模型「你要按步骤怎么做」
- **声明式 Agent 定义**：告诉 runtime「这个 Agent 的接口、约束、IO 契约是什么」，**由运行时负责调度和流程控制**，大模型只负责承载业务逻辑。

👉 我之前的做法，是把「声明的契约、规则、Grilling 逻辑」全部写进提示词里给大模型自己遵守。
这是**用提示词模拟声明式能力**，并不是真正的声明式 Agent：
真正声明式的，**校验、Grilling、资产解析、确认流程、NextSuggestion 提取，是 runtime 代码层的能力，不是靠大模型自觉遵守提示词**。
大模型一旦失忆、漏步骤、跳过 Grilling、忘记展示输入清单，整套就崩了 —— 这就是提示词方案的软肋。

---

## 那你的表述问题在哪（很微小，只是容易撞歧义）

你原话：**声明式 Agent**
这个短语本身是歧义短语：

1. 狭义（Copilot 社区）：声明式配置文件定义的 Copilot Agent（`.agent.md`）✅ 也是声明式
2. 广义（Agent framework / LangGraph / Agentic 架构圈）：基于 IO 契约、schema、约束声明，由 runtime 驱动的 Agent ✅ 也是声明式

你前面紧跟着的需求：

>
> 输入输出提示给用户、用户确认环节、输出里给出下一个 Agent 以及要做什么、需求模糊时 Grilling、可复制模板、全部同意

这套东西，天然偏向 **runtime 驱动的声明式 Agent 模型（语境 B）**，而不是 Copilot md Agent（语境 A）。
但我第一眼看到「声明式 Agent」，优先匹配了我最近接触最多的 Copilot 那套，直接选了载体 A，把整套契约塞到提示词里。

>
> 所以不是你说错了；是这个术语有两个流行版本，我优先选了其中一个载体假设，没有先和你对齐：
> “你说的声明式 Agent，是【Copilot md 声明配置 Agent】，还是【一套独立 runtime 读取 Schema 契约驱动的声明式 Agent】？”

---

## 两种方案的对比，帮你选回你想要的路线

### 方案 1（我前面一直在做的：提示词内声明，Copilot Agent）

- 载体：`.github/agents/xxx.agent.md`
- 运行主体：Copilot
- 规则、Grilling、IO 契约、资产解析：全部写在 system prompt 里，靠大模型自律执行
- 优点：开箱即用，不需要自己写 Runtime；直接在 Copilot 里跑
- 缺点：弱约束，模型容易飘、漏步骤；没有强 Schema 校验；没有真正的机器可读契约

### 方案 2（你原本想要的：真正声明式 Agent，契约交给 Runtime）

- 载体：**不是 md 提示词文件**，而是一份结构化声明文件（yaml/json）
- 声明内容是一等公民：
```
agent: ProductManager
input_schema: ...
output_schema: ...
grilling_strategy:
  enabled: true
  collect_all_questions_at_once: true
next_suggestion_enabled: true
confirmation: require_user_confirm_before_asset_persist: true
```
- Runtime（你自己写的程序）负责：
    1. 读取这份声明
    2. 收集输入资产，按 schema 校验
    3. 校验失败 → runtime 自动生成 Grilling 块（不是模型自己写什么时候要 Grilling）
    4. 拿到模型输出 → runtime 提取 asset 块、提取 NextSuggestion、校验输出结构
    5. 执行用户确认、资产入库
- 大模型只负责业务推理（写 PRD、做业务分析），**流程控制不归模型管**
- 优点：强约束、稳定、可扩展；模型不会随便跳过流程；契约可以被其他程序消费
- 缺点：你要自己实现一套轻量 Runtime（解析声明、Grilling 流程、资产池、确认流转）

## 现在我们可以重新对齐

你可以选一句：

1. 我想要的就是【方案 2】：声明文件是 Schema / 契约，单独写 Runtime 驱动，大模型只干活，流程由代码管控，之前的 md 只是临时提示词模拟，不是最终形态
2. 我其实还是想用 Copilot 的 `.agent.md`，只是我口中的 “声明式 Agent” 单纯指：在 agent 文件里声明清楚 IO 契约、规则，让 Copilot Agent 按这套契约跑（接受模型自律的弱点）

如果你是方案 2，我们就可以：

- 放弃把所有逻辑塞进 md 提示词
- 设计一份 yaml 格式的 Agent 声明规范（声明 IO schema、Grilling 开关、确认策略、默认 next suggestion）
- 同时配套设计 runtime 的模块（资产池、grilling 生成器、确认控制器）
- 模型侧只给精简的 system prompt，不再塞一大堆流程规则

你想往哪个方向走？

昨天 23:45

不是的，我要的就是.github/agents/xxx.agent.md，我一开始称呼它为声明式 Agent，你弄错了，那么我们通常怎么表达这类 .github/agents/xxx.agent.MD Agent，或是有什么更准确的表述，能不让别人误解

# 官方标准叫法（来自 GitHub 文档）

**GitHub 官方名称：Copilot Custom Agent，文档里也叫 Agent Profile**GitHub Doc...
文件：`.github/agents/*.agent.md`

>
> 英文标准表述：**GitHub Copilot Custom Agent（Agent Profile）**
> 中文标准表述：**GitHub Copilot 自定义 Agent（Agent 配置文件 / Agent 描述文件）**

关键点：

>
> `declarative agent / 声明式 Agent` 这个词**在行业里被 Microsoft 365 Copilot 抢注了**，特指微软 365 那套 JSON manifest 的 Agent，**不是 GitHub Copilot 的 .agent.md**，这就是本次误解的根源。
> 你用 “声明式 Agent” 去描述 `.agent.md`，技术圈里听到第一反应优先跳到 M365 Copilot，而不是 GitHub Copilot。

---

## 推荐几套话术，按场景选用，杜绝歧义

### ✅ 最稳妥、对外沟通（和别人、团队沟通首选，不会被误解）

1. 中文：**GitHub Copilot 自定义 Agent，定义在仓库 `.github/agents/xxx.agent.md`**
2. 英文：**GitHub Copilot Custom Agent (agent profile in `.github/agents/*.agent.md`)**

>
> 一句话示例：
> 我这套是 GitHub Copilot 自定义 Agent，放在仓库 `.github/agents/` 下面，每个角色一个 `.agent.md` 文件，内部约定 IO 契约、Grilling 澄清、资产输出、NextSuggestion 流转规则。

### ✅ 如果你还是想用 “声明式” 三个字（适合内部、你自己体系里，要加限定前缀）

>
> 不要单独说「声明式 Agent」，要带上限定：
> **GitHub Copilot 自定义声明式 Agent Profile**
> 并且第一次介绍时补充一句：
>
>
> >
> > 注意：不是 Microsoft 365 Copilot 的 declarative agent，是 GitHub Copilot 仓库内 `.agent.md` 形式的自定义 Agent。

>
> 为什么不推荐裸用：
> M365 的 declarative agent 是一个固定专有名词，有独立 schema、manifest、发布流程，和 GitHub Copilot 的 agent.md 完全两套东西，是最高概率撞车点。

### ✅ 极简内部简称（团队内部大家对齐之后）

- Copilot Agent Profile
- repo-level Copilot Agent（仓库级 Copilot Agent）

---

## 对比一句话区分，方便你自己记

- **Microsoft 365 Copilot Declarative Agent（声明式 Agent）**：JSON manifest，跑在 Teams/Office，微软编排器，专有名词
- **GitHub Copilot Custom Agent / Agent Profile**：`.github/agents/*.agent.md`，YAML frontmatter + 提示词，跑在 IDE / Copilot Chat / Copilot CLI，这就是你正在做的这套GitHub Doc...

---

## 顺带：你之前这套设计的准确描述（可以直接放进文档开场白）

>
> 本仓库使用 **GitHub Copilot Custom Agent（Agent Profile）**，文件存放于 `.github/agents/`，每个角色独立 `.agent.md`。
> Agent 文件内部声明角色身份、工具权限、输入输出契约、Grilling 澄清策略、产出确认规则、下一步推荐策略；
> 由 Copilot 大模型按文件内约定的规则执行，不是自研 runtime 调度的外部 Agent 框架。

如果你愿意，我可以把这句开场白加到前面 6 份 agent.md 文件的头部注释里，以后别人打开就不会混淆。

昨天 23:49

不只是 Copilot ，其它 AI 工具也会有类似 .xxx/agents/xxx.agent.MD，怎么统一称呼它们？

# 核心结论

先把容易冲突的专有名词划清，再给你一套分层命名方案：

>
> `declarative agent` 是**微软 M365 Copilot 的固定专有名词**，尽量不要拿来泛指这类基于 Markdown 配置文件的 Agent；
> 这类通用形态：**文件承载、前端 YAML 头元信息 + 系统提示词本体、放在仓库约定目录、可被各类 Agent 工具加载解析的 Agent 定义文件**，推荐统称：
>
>
> ✅ **Agent Profile（Agent 配置描述文件）** —— 首选通用术语，跨工具无歧义
> ✅ 全称：**File-based Agent Profile（基于文件的 Agent Profile）**

>
> 一句话定义：
> File-based Agent Profile：以独立文件（一般是 `.agent.md`）存放，头部带元信息（名称、描述、工具、权限）、主体为系统提示词，遵循目录约定（如 `.xxx/agents/`），供 AI Agent 加载器读取实例化 Agent 的配置文件。

---

## 可选同义词（按推荐优先级排序）

1. **Agent Profile（首选）**
   来源：GitHub Copilot 官方文档就是把 `.agent.md` 叫做 Agent Profile；
   好处：已经有现成认知，又**不绑定 GitHub/Copilot**；你可以扩展定义为通用概念，不限于 Copilot。
   英文：Agent Profile
   中文：Agent 配置描述 / Agent 档案

示例话术：

>
> 我这套是 **Agent Profile**，统一放在仓库 `./agents/` 目录下，文件后缀约定为 `.agent.md`，兼容多款支持加载 Profile 的 AI 工具。
2. **Agent Manifest（备选，慎用）**
   Manifest 本意是清单；但大量框架（M365、Bot、插件）用 manifest 指代 JSON/YAML 清单，**默认不带大段提示词正文**，容易被理解成纯结构化清单、没有 system prompt。

>
> 适合：只有元数据、无大段提示词的；不太适合你这种 frontmatter + 长篇 prompt 的 md 文件。
3. **Agent Spec（Agent 规格）**
   偏技术内部叫法，适合团队内部，对外弱一点；更偏向 IO / 行为契约，不是载体。
4. **Agent Prompt Profile（更精准，长一点）**
   强调：它本质是**带元信息的提示词 Profile**，适合需要特意区分 “纯结构化配置” 和 “带系统提示词” 的场景。

❌ 不推荐泛用：

- Declarative Agent：绑定 M365 专有名词，一用就容易跑偏；
- Custom Agent：太泛，网页 UI 里新建一个 Agent 也叫 Custom Agent，看不出是文件形态；
- Agent Definition：也泛，可以是代码定义、UI 定义，不一定是文件 md。

---

## 你自己内部的术语体系（可以直接固化到你的文档）

你可以在项目 README 里写一段术语规约，一劳永逸避免歧义，我给你一段可直接复制：

>
> ### 术语约定
>
>
> 【Agent Profile】
> 泛指存放在仓库约定目录（如 `.github/agents/`、`.opencode/agents/`）、命名为 `*.agent.md` 的 Agent 描述文件；
> 文件结构固定为：YAML frontmatter（元信息：name /description/tools /permission 等） + 系统提示词正文；
> 可被多种兼容工具加载实例化 Agent，不限于 GitHub Copilot。
>
>
> 区分：
>
>
> - M365 Declarative Agent：微软 365 生态专有形态，JSON manifest，不属于本项目所说的 Agent Profile；
> - Inline Agent：在对话里临时定义、不落地成文件的 Agent；
> - Code Agent：代码编写的 Agent（LangGraph 等代码编排）。

---

## 配套：一句话不同场景的表达模板

1. 对内团队沟通（最简）

>
> 每个角色对应一份 Agent Profile，放在 agents 目录，`.agent.md` 格式。

2. 和外部懂 GitHub Copilot 的人沟通（避免对方以为仅限 Copilot）

>
> 我采用了类似 GitHub Copilot Agent Profile 的文件规范，`*.agent.md`，YAML 头 + 提示词，自己做了加载器，支持在多款 AI 工具里加载。

3. 写架构文档（严谨版）

>
> 本系统采用 **File-based Agent Profile** 规范：以 Markdown 文件作为 Agent 载体，约定目录与文件后缀，元信息存放在 Frontmatter，主体为系统提示词，兼容多款 Agent 运行时。

---

## 顺带一个小优化建议（适配你现在的文件）

如果你想让这套规范更通用，不再强绑定 `.github`，可以：

- 把目录做成可配置基准目录：默认 `.opencode/agents/`（你本地 F 盘就是这个路径）
- 在 Profile 的 frontmatter 里，**不要写平台专属字段**（比如不要写 Copilot 专属的字段），只保留通用元字段：`name, description, tools, permissionMode`
  这样它就是一份**通用 Agent Profile**，既能在 Copilot 跑，也能在你自己的 Agent 加载器里跑。

如果你想要，我可以帮你写一份独立的 `AGENT-PROFILE-SPEC.md`，作为这个通用 `.agent.md` 格式的正式规范文档。

昨天 23:50

写一份独立的 AGENT-PROFILE-SPEC.md

我将为你撰写一份**通用、跨平台、行业可落地**的《AGENT-PROFILE-SPEC》正式规范，彻底定义你这套 `*.agent.md` 体系，完全脱离 GitHub Copilot 私有属性，成为你个人 / 团队通用标准，解决所有术语歧义。

# AGENT-PROFILE-SPEC 通用 Agent Profile 文件规范

## 版本：v1.0 | 稳定通用版

## 适用范围：跨 AI 工具、跨运行时、可移植的文件式 Agent 定义标准

## 0. 前言 & 术语统一（解决所有歧义）

### 0.1 核心定义

本规范定义 **File-based Agent Profile（文件式 Agent 配置档案）**
统一称谓：**Agent Profile**

>
> 严格区别行业歧义名词：
>
>
> 1. ❌ **声明式 Agent（Declarative Agent）**：Microsoft 365 Copilot 专有名词，禁止混用
> 2. ❌ **Copilot Custom Agent**：GitHub 私有实现，本规范**不绑定任何平台**
> 3. ✅ **Agent Profile**：本体系唯一标准通用名称

### 0.2 规范定位

Agent Profile 是一种**纯文件、声明式、无代码、可移植**的 AI 角色定义标准：

- 不依赖编程语言
- 不依赖指定 AI 平台
- 可被任意兼容运行时解析加载
- 统一团队所有角色 Agent 的 IO 契约、交互行为、工作范式

### 0.3 核心特征

- **文件即实体**：一个 `.agent.md` 文件 = 一个标准化 AI 角色
- **结构分层**：Frontmatter 元数据 + 系统规则 + 业务契约 + 输出模板
- **行为标准化**：统一 Grilling 澄清、用户确认、资产 IO、下一步推荐
- **生态通用**：兼容 GitHub Copilot、本地 AI 工具、私有 Agent 运行时

---

## 1. 文件规范

### 1.1 目录约定

所有 Agent Profile 统一存放于项目固定目录：

```
/.opencode/agents/
# 兼容别名（适配Copilot原生）
/.github/agents/
```

### 1.2 文件命名规范

- 后缀固定：`.agent.md`
- 命名格式：`大驼峰角色名.agent.md`
- 示例：
    - `ProductManager.agent.md`
    - `BackendEngineer.agent.md`
    - `Architect.agent.md`

### 1.3 文件结构总纲（固定四层结构，强制）

所有 Agent Profile **必须严格遵循四层结构，顺序不可打乱**：

1. **Frontmatter 元数据层**（机器可读、运行时配置）
2. **全局通用规则层**（全角色统一行为范式）
3. **角色专属契约层**（输入契约 + 输出契约）
4. **角色行为扩展层**（专属能力、默认推荐逻辑）

---

## 2. 第一层：Frontmatter 元数据规范（通用可移植）

### 2.1 通用固定字段（跨平台通用，无平台私有属性）

```
---
name: 角色唯一名称（英文大驼峰）
description: 角色一句话定位描述
tools: [允许调用的工具列表]
permissionMode: plan
---
```

### 2.2 字段释义

表格

| 字段 | 规则 | 说明 |
| --- | --- | --- |
| name | 唯一、英文大驼峰 | Agent 唯一标识，用于资产溯源、Next 推荐 |
| description | 简短精准 | 角色定位，供 AI 和人识别用途 |
| tools | 数组 | 该角色允许使用的文件 / 命令 / 读写工具 |
| permissionMode | 固定 plan | 所有角色统一**先计划、后执行、需确认** |

>
> permissionMode: `plan` 为本规范**强制标准**
> 含义：所有修改、执行类动作，必须先展示计划、等待用户确认，禁止自主执行

---

## 3. 第二层：全局统一行为规范（全角色强制一致）

所有 Agent Profile **必须包含完全一致的全局规则**，保证团队所有 Agent 行为范式统一。

包含四大固定模块：

1. **输入契约校验流程**（自动扫描资产、展示输入清单）
2. **Grilling 结构化澄清流程**（Matt Pocock 标准：批量确认、可复制模板、全部同意）
3. **三段式标准化输出**（资产 + 下一步推荐 + 用户确认）
4. **核心架构约束**（资产隔离、网状依赖、只读不篡改）

### 3.1 固定全局规则文本（所有角色复用）

```
# ========== 全局最高优先级规则（全角色统一终版） ==========
## 1. 输入契约执行流程（启动第一步，必须先做）
1. 扫描对话上下文 + 项目Agent资产目录，收集本Agent输入契约定义的必填/可选资产。
2. 整理【本次输入摘要】展示给用户：
   - ✅ 已加载资产列表：asset_id、asset_type、摘要
   - ❌ 缺失必填资产 / 待决策点：立刻触发 Grilling 澄清流程
3. 所有必填资产就绪、所有模糊点、决策点全部闭环后，询问用户：是否确认启动本次任务。
4. 用户确认后才开始工作，禁止私自启动。
> 输入来源不限：Agent产出资产、项目存量文档、用户口述需求、历史迭代资料。

## 2. Grilling 澄清流程（标准化批量确认机制）
当出现以下任意情况，**禁止零散提问、禁止脑补、禁止跳过**：
- 需求描述模糊、缺少关键约束
- 必填信息缺失
- 存在多种可行方案需要用户决策
- 验收标准、目标、范围不明确

必须输出标准化可复制问卷：
> 🔍 需要确认事项（Grilling）
> 下面是可复制应答模板，已附带最优建议回答
> 你可以：
> ① 直接回复：`全部同意` 一键采纳所有建议
> ② 复制整块 JSON 修改任意项后回复

```grilling
{
  "confirm_items": [
    {
      "id": "",
      "question": "",
      "suggested_answer": "",
      "type": "text | single_choice | boolean",
      "options": []
    }
  ],
  "notes": ""
}
```

### Grilling 强制规则

1. 一次性列出**所有待确认项**，不分批、不分次。
2. 每项必须给出合理、专业的默认建议值。
3. 用户回复「全部同意」→ 自动采纳所有建议。
4. 用户修改 JSON → 以用户版本为准。
5. 澄清完毕汇总结果给用户复核，再继续任务。

## 3. 标准化输出三段式（固定顺序不可乱）

任务完成后必须严格按以下三部分输出：

### A. 标准结构化资产（机器可读、跨 Agent 消费）

```
{
  "asset_type": "",
  "producer": "",
  "asset_id": "",
  "summary": "",
  "depends_on_asset_ids": []
}
```

【角色专属结构化 JSON 资产内容】

### B. NextSuggestion 下一步推荐（人读调度建议，非强制流转）

>
> 📌 推荐后续 Agent：xxx
> 🎯 期望执行任务：清晰、具体、明确需要完成的工作、依赖资产、重点关注点

### C. 用户归档确认

>
> 是否确认接纳并归档本份资产？
> 可选：【确认资产生效进入资产池】 / 【修改内容】 / 【仅保留对话不归档】

## 4. 核心架构约束（体系基石）

1. Asset、Grilling、NextSuggestion **完全隔离**
    - 其他 Agent 仅解析 `asset` 块（正式业务资产）
    - Grilling 仅为临时交互，不入库
    - NextSuggestion 仅为人的辅助调度决策
2. 无固定流水线，**网状按需依赖**，不强制执行顺序
3. 一份资产可被任意多个下游 Agent 消费，支持一对多复用
4. 绝不修改存量资产，只生产新资产、新迭代版本

```

---

## 4. 第三层：角色专属IO契约（每个角色独立唯一）
### 4.1 固定双契约结构
每个Agent必须定义：
1. **Input Contract 输入契约**
   - 必填资产列表
   - 可选资产列表
   - 不限制信息来源（存量文件/用户输入/其他Agent输出）

2. **Output Contract 输出契约**
   - 唯一固定 `asset_type`
   - 固定不可变 JSON Schema 结构
   - 产出永久标准化，供全体系Agent解析消费

### 4.2 设计原则
- **输入只定义「需要什么」，不定义「谁提供」**（适配网状架构）
- **输出唯一Schema终身固定**，保证跨Agent兼容
- 所有业务产出必须结构化，禁止自由文本作为正式资产

---

## 5. 第四层：角色专属行为扩展
每个角色尾部可配置：
- 默认 NextSuggestion 推荐逻辑
- 角色专属工作侧重点
- 行业默认最佳实践

仅作为辅助人读建议，**不影响机器解析逻辑**。

---

## 6. 体系核心资产规范（跨文件通用）
### 6.1 资产块标准
所有正式业务资产统一使用：```` ```asset ```` 代码块包裹
所有Agent解析输入时，**只识别该块内容**，忽略所有自然语言、提问、建议文本。

### 6.2 Grilling块标准
临时交互问卷统一使用：```` ```grilling ```` 代码块包裹
仅用于人机澄清，**不进入资产池、不被其他Agent消费**。

### 6.3 调度建议标准
NextSuggestion 纯文本区块，仅为人的调度参考，**无机器解析逻辑**。

---

## 7. 体系架构特性（本规范核心优势）
1. **无固定流水线**
   摒弃线性A→B→C强制流转，完全按需依赖、网状协作
2. **多源输入兼容**
   支持：用户输入、历史Agent资产、项目存量文档、迭代更新
3. **低门槛人机交互**
   统一「全部同意」极简操作，降低用户决策成本
4. **强标准化、弱耦合**
   角色之间只通过结构化资产通信，完全解耦
5. **跨平台可移植**
   脱离Copilot私有约束，可用于任意支持md profile的AI运行时

---

## 8. 标准角色资产类型清单（内置生态）
| 角色 | 唯一asset_type |
| :--- | :--- |
| ProductManager | StructuredPRD |
| BusinessAnalyst | BusinessSpec |
| Architect | ArchitectureDesign |
| BackendEngineer | BackendTechDesign |
| Tester | TestPlanAndCase |
| DeliveryManager | DeliveryPlan |

---

## 9. 版本迭代规则
1. 本规范版本号写入文件头部
2. 资产Schema**大版本不兼容变更需升级规范版本**
3. 行为规则统一更新，所有Agent Profile同步对齐
4. 永远保持：结构四层不变、三块隔离不变、Grilling机制不变

---

## 10. 最终对外标准话术（彻底杜绝歧义）
### 标准中文表述
> 本项目采用 **AGENT-PROFILE-SPEC 通用文件式Agent规范**，
> 以 `.agent.md` 文件作为标准化AI角色载体，
> 统一所有角色的输入输出契约、人机交互规则、资产生成范式，
> 属于跨平台可移植的自定义Agent体系，非平台私有能力。

### 标准英文表述
> This project follows **AGENT-PROFILE-SPEC**, a universal file-based Agent profile standard.
> All AI roles are defined via `.agent.md` with standardized IO contracts,
> grilling interaction and structured asset delivery, cross-platform compatible.
```

这份 **AGENT-PROFILE-SPEC.md** 是你的**私有行业标准终版规范**，彻底解决术语混淆问题，所有 `.agent.md` 文件都严格对齐该规范。

需要我帮你**把 6 个角色的 Agent 文件全部同步标注规范版本号**，让整套体系完全统一合规吗？