---
title: "Matt Pocock Skills 完整使用指南与最佳实践"
source: "https://www.cnblogs.com/Boy-boy/p/21844276"
author:
  - "[[怀恋小时候]]"
published: 2026-07-23
created: 2026-08-12
description: "Matt Pocock Skills 完整使用指南与最佳实践 ⚠️ 版本适用声明 适用技能版本：mattpocock/skills (基于 docs/agents/ 懒加载与 Multi-Context 架构) 测试适配环境：Claude Code（VSCode 插件 / 独立 CLI 均可）、Co"
tags:
  - "clippings"
---
> ⚠️ **版本适用声明**
> 
> - **适用技能版本** ： `mattpocock/skills` (基于 `docs/agents/` 懒加载与 Multi-Context 架构)
> - **测试适配环境** ：Claude Code（VSCode 插件 / 独立 CLI 均可）、Codex CLI
> - **示例项目** ：StockAnalyzer
> - **官方仓库** ： [github.com/mattpocock/skills](https://github.com/mattpocock/skills)
> - **安装工具** ： [vercel-labs/skills](https://github.com/vercel-labs/skills) (即下文 `npx skills` 命令)  
> 	*注：AI 工具链迭代频繁，若后续官方有重大变更，请以官方仓库最新文档为准。*

---

## 目录

## 1\. 简介

`mattpocock/skills` 是由 TypeScript 专家 Matt Pocock 开源的一套面向 **AI 编码助手（AI Coding Agents）** 的标准化工程技能集（Agent Skills）。

该技能集旨在摒弃盲目代码生成的“Vibe Coding”模式，为 AI Agent 引入工程化约束与最佳实践，提供涵盖 **需求访谈、技术规格定义、测试驱动开发（TDD）、缺陷诊断及 GitHub Issue 自动化管理** 的全流程标准规范。

### 核心特性

- **知识库防污染沉淀** ：通过严格的领域字典（Glossary）与多上下文映射（ `CONTEXT-MAP.md` ），实现跨需求的业务概念统一，同时隔离代码细节。
- **Issue 跟踪集成** ：原生集成 GitHub Issues（依赖 `gh` CLI），实现自动创建与标签状态流转。
- **多 Agent 跨平台兼容** ：完美兼容 Claude Code、Codex CLI、Cursor、Cline 等支持 Agent Skills 规范的客户端。
- **命令调起规范** ：
	- **Claude Code（VSCode / CLI）** ：通过斜杠调用，如 `/grill-with-docs` 。
		- **Codex CLI** ：不带斜杠调用，如 `grill-with-docs` 。

---

## 2\. 环境前置要求

在正式配置前，请确保本地开发环境满足以下依赖条件：

1. **Node.js 运行环境** ：建议版本 ≥ 18.0.0（用于通过 `npx` 执行 Agent Skills 安装程序）。
	```sh
	node -v
	```
2. **GitHub CLI 工具（可选，使用工单功能必选）** ：安装 [GitHub CLI (`gh`)](https://cli.github.com/) 并完成身份认证。
	```sh
	# 检查登录状态
	gh auth status
	# 未登录时执行交互式登录
	gh auth login
	```
3. **IDE / 插件准备** ：VSCode 需提前安装并激活 Claude Code 插件。

---

## 3\. 安装与配置步骤

### 3.1 全局安装技能（推荐 Symlink 模式）

打开 PowerShell 或 Terminal 执行：

```sh
npx skills@latest add mattpocock/skills
```

> 🪟 **Windows 用户注意** ：如果选择 Symlink 模式,Windows 创建符号链接需要管理员权限或"开发者模式"。二选一:
> 
> - 用 **管理员权限** 的 PowerShell 执行上述命令,或
> - 在系统设置里开启"开发者模式"(设置 → 更新与安全 → 开发者选项),之后普通终端也能创建软链接
> 
> macOS/Linux 无此问题,普通终端直接执行即可。

在交互式选项中建议选择：

```
> Symlink (Recommended)
```

> 💡 **安装模式说明** ：
> 
> - **Symlink（推荐）** ：多个客户端共享同一份源码，官方更新后可一键同步全局生效。
> - **Copy to all agents** ：将技能文件完整拷贝至各 Agent 规则目录。仅在企业受控终端限制软链接权限时使用。

安装后技能文件默认存储于用户家目录下的 `.agents` 路径：

```
C:\Users\<Your-Username>\.agents\skills\   # Windows
~/.agents/skills/                          # macOS/Linux
```

### 3.2 为 Codex CLI 绑定技能（可选）

```sh
npx skills@latest add mattpocock/skills -a codex
```

### 3.3 技能更新与卸载

```sh
# 拉取并更新技能至最新版
npx skills@latest update mattpocock/skills

# 卸载技能
npx skills@latest remove mattpocock/skills
```

---

## 4\. 项目初始化与知识库隔离架构【核心】

> 📌 **规则** ：每一个独立的代码仓库在引入该技能集时， **仅需且必须执行一次** `/setup-matt-pocock-skills` 。

### 4.1 初始化生成的目录结构

执行初始化后,AI Agent 会在项目根目录与 `docs/agents/` 建立标准配置。

**Before(裸仓库)→ After(跑完 `/setup-matt-pocock-skills`):**

```
# Before                              # After
StockAnalyzer/                        StockAnalyzer/
├─ src/                               ├─ src/
├─ package.json                       ├─ package.json
└─ README.md                          ├─ README.md
                                      ├─ CLAUDE.md                 ← 新增(或追加 ## Agent skills 区块)
                                      └─ docs/
                                         └─ agents/                ← 新增目录
                                            ├─ issue-tracker.md    ← 新增
                                            ├─ triage-labels.md    ← 新增(仅 triage 已安装时)
                                            └─ domain.md           ← 新增
```

**各文件长什么样(节选):**

- `CLAUDE.md` 里会追加一个块:
	```markdown
	## Agent skills
	### Issue tracker
	Issues live in GitHub Issues. See \`docs/agents/issue-tracker.md\`.
	### Triage labels
	Using default vocabulary. See \`docs/agents/triage-labels.md\`.
	### Domain docs
	single-context. See \`docs/agents/domain.md\`.
	```
- `docs/agents/issue-tracker.md` ——告诉后续 skills"issue 存哪、用什么 CLI 操作"
- `docs/agents/triage-labels.md` ——五种标签名映射(`needs-triage` / `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix`)
- `docs/agents/domain.md` ——告诉后续 skills" `CONTEXT.md` 和 ADR 存哪、什么时候读"

> 💡 **注意:** 此步骤 **不会** 创建 `CONTEXT.md` 和 `docs/adr/`,那是懒加载的——见 4.2。

---

### 4.2 防污染机制与 Multi-Context 实现路径

很多开发者担心：“连续开发多个需求后， `CONTEXT.md` 会不会变成充斥着乱七八糟代码细节的污染源？”

**答案是：绝对不会。** 框架通过以下三种机制保障知识库的高纯度：

#### 1\. 懒加载（Lazy Creation）

初始化完成后 **不会强制生成空文件** 。只有当你第一次调用 `/grill-with-docs` 且真正收炼出明确的 **领域术语或业务约束** 时，Agent 才会创建 `CONTEXT.md` 或 `docs/adr/` 。

**具体示例:** 假设你第一次跑 `/grill-with-docs` 澄清"K 线数据导入"需求,拷问过程中定义了「K 线」「日线」「预告开播」这些术语——AI 会自动生成:

```
StockAnalyzer/
├─ CONTEXT.md                     ← 新增,内容如下
└─ docs/
   ├─ agents/ (略)
   └─ adr/                         ← 新增(如果有架构决策)
      └─ 0001-csv-import-format.md ← 新增(仅当出现"难以逆转 + 需要理由 + 有真实取舍"三要素时)
```

`CONTEXT.md` 样例:

```markdown
# StockAnalyzer 领域词典

## K 线 (K-Line)
一根 K 线代表一个交易周期(默认日线)内的四个价格:开盘、收盘、最高、最低。
日线以自然日切分。

## 预告开播 vs 真实开播
- 预告开播:直播间预设的开播时间点(数据配置)
- 真实开播:主播实际推流的时刻
业务判断"是否开播"时需明确指哪一种。
```

> 📌 **关键:** `CONTEXT.md` 只存 **业务词汇与规则约束**,不存代码/接口/实现细节。ADR 只在"难以逆转、需要写下理由、有真实取舍"三个条件全部满足时才生成——不是每次都产。

#### 2\. 统一领域字典（Use the Glossary's Vocabulary）

`CONTEXT.md` 存放的是 **跨需求共享的业务名词与不变量** （例如定义什么叫“K 线”、什么叫“最大回撤算法”），而 **非特定需求的实现细节** 。AI 在后续任何开发中，都必须遵守该字典词汇，防止用词漂移（Vocabulary Drift）。

#### 3\. 大型/多模块项目的物理隔离（Multi-Context 实现机制）

对于大型项目或 Monorepo，框架支持通过 `CONTEXT-MAP.md` 划定模块边界。 **Multi-Context 的建立是“半自动”的（AI 具备构建能力，但需要你主动触发或授权）** 。

##### 目录结构对比

- **小型/单上下文项目（单文件）** ：
	```
	/
	├── CONTEXT.md               # 统一业务词典
	├── docs/adr/                # 全局架构决策 (ADRs)
	└── src/
	```
- **大型/多上下文项目（物理隔离）** ：
	```
	/
	├── CONTEXT-MAP.md           # 路由地图：指定各子系统的上下文映射
	├── docs/adr/                # 系统级总决策
	└── src/
	    ├── ordering/            # 订单模块
	    │   ├── CONTEXT.md       # 仅涵盖订单领域的业务术语
	    │   └── docs/adr/        # 模块专属 ADR
	    └── billing/             # 计费模块
	        ├── CONTEXT.md       # 仅涵盖计费领域的业务术语
	        └── docs/adr/
	```

##### 如何触发与落地 Multi-Context 拆分？

不需要开发者手动新建文件，可以通过以下两种方式实现：

1. **方式 A：通过自然语言指令显式触发（推荐）**  
	当发现项目包含多个独立子系统时，直接在 Claude Code 中发送指令：
	> “这个项目模块较多，请帮我重构为 Multi-Context 知识库架构，在根目录生成 `CONTEXT-MAP.md` ，并在 `src/` 各子模块下建立独立上下文。”
	Agent 会扫描代码库、给出 **它认为的** 模块边界(Bounded Context)划分方案,以及对应的 `CONTEXT-MAP.md` 与各子模块 `CONTEXT.md` 草稿。  
	⚠️ **实际使用注意**:AI 的模块划分是"建议"不是"事实",生成后 **必须人工 review** ——尤其是模块边界不清晰的项目,AI 可能划得过细或过粗,需要你手工调整后再采纳。
2. **方式 B：通过架构优化命令被动触发**  
	运行 `/improve-codebase-architecture` 命令时，Agent 如果扫描到多模块架构，会主动提示：
	> *“检测到多业务模块，是否需要生成 `CONTEXT-MAP.md` 以隔离子领域的上下文？”*  
	> 输入 `Yes` 后,Agent 会给出重构方案,同样需要人工确认后落地。

##### 拆分后的日常使用

拆分完成后，开发者在日常开发中 **无需进行任何额外切换** 。例如编写计费需求时：

```
/grill-with-docs
开发 billing 模块的发票导出功能...
```

Agent 会根据 `docs/agents/domain.md` 中的路由逻辑，自动定位并 **仅读取** `src/billing/CONTEXT.md` ，彻底在物理层面隔离了 `ordering` 等其他模块的上下文干扰。

---

## 5\. 核心命令清单 & 使用规范

> 🧭 **万能导航** ：如果不确定当前场景该使用哪个技能，输入 `/ask-matt` 并描述你的诉求，Agent 会自动为你推荐最优的技能工作流。

| 命令 | 作用说明 | 推荐优先级 | 生成物 / 副作用 |
| --- | --- | --- | --- |
| `/grilling <描述>` | **轻量版需求访谈**,只拷问、不写文档 | ⭐⭐⭐⭐⭐ | **无文件产出** ——只在对话里问答 |
| `/grill-with-docs <描述>` | 深度需求访谈,自动更新领域词典 | ⭐⭐⭐⭐⭐ | 📄 新增/更新 `CONTEXT.md` (有新术语时)   📄 新增 `docs/adr/NNNN-xxx.md` (仅当出现难以逆转的架构决策) |
| `/to-spec` | 将对话总结为技术规格并发布到 issue 追踪器 | ⭐⭐⭐⭐ | 🎫 GitHub Issue(远程模式)   📄 `.scratch/<feature>/spec.md` (本地模式) |
| `/implement` | 按 spec/ticket 实现代码,自带完工纪律 | ⭐⭐⭐⭐ | 📝 修改源码文件(`src/**`)   📝 新增/修改测试文件   🔀 一个 `git commit` |
| `/tdd` | 严格 TDD:先写 failing test 再写实现 | ⭐⭐⭐⭐⭐ | 📝 交替新增测试文件与实现文件   🔀 通常配合多次小 commit |
| `/prototype` | 快速原型验证 | ⭐⭐⭐ | 📝 新增原型代码文件(标注"prototype"字样)   🌿 建议提交到 throwaway 分支 |
| `/diagnosing-bugs` | 标准化缺陷排查 | ⭐⭐⭐⭐⭐ | **主要为对话输出** (假设+验证方案)   📝 可能新增回归测试文件 |
| `/code-review` | 工程规范与安全性代码审查 | ⭐⭐⭐⭐ | **无文件产出** ——评审报告在对话里 |
| `/to-tickets` | 将方案拆解为工单 | ⭐⭐⭐ | 🎫 多个 GitHub Issues(远程)或 📄 `.scratch/<feature>/*.md` 一 ticket 一文件(本地) |
| `/triage` | 批量管理 Issue 状态 | ⭐⭐ | 🎫 修改远程 issue 的标签/评论 |
| `/handoff` | 会话摘要,便于新窗口继续 | ⭐⭐⭐ | 📄 **写入 OS 临时目录** (不污染工作区),Windows 一般在 `%TEMP%\claude-handoff-*.md` |
| `/research` | 技术调研 | ⭐⭐⭐ | 📄 在仓库既有的调研文档目录下新增 Markdown(带引用来源);无固定目录时会说明放哪 |
| `/resolving-merge-conflicts` | 辅助解决合并冲突 | ⭐⭐ | 📝 修改冲突文件   🔀 完成 merge/rebase commit |
| `/improve-codebase-architecture` | 架构审查与优化建议 | ⭐ | 📄 **HTML 报告写入 OS 临时目录** (不入库),自动打开浏览器 |
| `/ask-matt` | 技能路由 | ⭐⭐⭐⭐⭐ | **无文件产出** ——只在对话里推荐 |

**图例:** 📄 = Markdown 文档 | 📝 = 源码 | 🎫 = 远程工单 | 🔀 = git commit | 🌿 = 独立分支

> 💡 **速记原则:**
> 
> - **动代码的**:`/implement` 、 `/tdd` 、 `/prototype` 、 `/resolving-merge-conflicts`
> - **写文档到仓库的**:`/grill-with-docs` (领域词典)、 `/research` (调研笔记)
> - **上传到工单系统的**:`/to-spec` 、 `/to-tickets` 、 `/triage`
> - **只在对话/临时目录的**:`/grilling` 、 `/code-review` 、 `/diagnosing-bugs` 、 `/ask-matt` 、 `/handoff` 、 `/improve-codebase-architecture`

---

### 📌 补充说明:关于命令表里的"远程 / 本地"

表格里 `/to-spec` 、 `/to-tickets` 、 `/triage` 三个命令都出现了 **"远程" vs "本地"** 两种产出形式。这是因为 skills 允许你选择"issue 存哪里"——跑 `/setup-matt-pocock-skills` 初始化时会问你:

> "这个项目的工单(issue / spec / ticket)存到哪?"
> 
> - **GitHub Issues** (远程模式)—— 用 `gh` CLI 操作 GitHub Issues
> - **GitLab Issues** (远程模式)—— 用 `glab` CLI 操作 GitLab Issues
> - **Local Markdown** (本地模式)—— 每个 issue 就是仓库里的一个 `.md` 文件,不依赖外部服务
> - **Other** (Jira / Linear 等)—— 自己描述工作流

你选的答案会写进 `docs/agents/issue-tracker.md`,后续所有 skills 都读这个配置来决定"发到哪"。

#### 两种模式下 /to-tickets 产出对比

**远程模式(选了 GitHub):** 调用 `gh issue create`,在你的 GitHub 仓库真实创建多个 issue。产出是网页上能看到的 issue,不落本地文件:

```
GitHub 上多出这些新 issue:
├─ #42  [csv-import] Parse CSV rows into K-line records
├─ #43  [csv-import] Validate field types and ranges       (blocked by #42)
├─ #44  [csv-import] Upsert batch with duplicate handling  (blocked by #43)
└─ #45  [csv-import] Import summary logger                 (blocked by #42)
```

**本地模式(选了 Local Markdown):** 在仓库里新建目录,一个 ticket 一个 `.md` 文件,全部落地在磁盘上:

```
.scratch/csv-import/               ← <feature-slug>,以特性名命名
├─ spec.md                         ← /to-spec 的产出放这里
└─ issues/                         ← 一堆 ticket 文件
   ├─ 01-parse-csv-rows.md
   ├─ 02-validate-fields.md
   ├─ 03-upsert-with-dedup.md
   └─ 04-import-summary-logger.md
```

每个 issue 文件内部长这样:

```markdown
# Parse CSV rows into K-line records

Status: ready-for-agent
Blocked by:

## Description
...ticket 内容...

## Comments
...后续讨论追加到这里...
```

#### 怎么选?

| 场景 | 推荐模式 |
| --- | --- |
| 团队协作、有 GitHub/GitLab 仓库、希望 issue 状态和讨论对所有人可见 | **GitHub / GitLab 远程模式** |
| 个人项目 / 不想开外部账号 / 想让 issue 跟代码一起进 Git 版本控制 | **本地模式** |
| 公司统一用 Jira / Linear / 飞书等 | **Other** ——初始化时详细描述工作流,skills 会尽力对齐 |

> 💡 **补充:** 无论选哪种模式,`/handoff` 、 `/improve-codebase-architecture` 生成的临时文件都 **始终** 放在 OS 临时目录,不受这里的模式选择影响——那是"会话性产出",不是"工单"。

---

## 6\. 典型场景实战工作流

### 场景 1:普通业务功能开发(数据导入 / 后台接口)

**推荐链路:** `/grill-with-docs` → `/to-spec` → `/implement` → `/code-review`

**每步的输入与产出对比:**

```
/grill-with-docs
开发 K 线历史数据导入模块:支持从 CSV 文件读取日线数据,校验时间、开盘价、收盘价、成交量字段,
存入数据库;重复日期数据自动覆盖,输出导入成功条数与异常日志。
```

**产出:** 若过程中出现新术语,自动在根目录新增/更新:

```
+ CONTEXT.md         (新增/追加"K 线""日线""导入批次"等术语定义)
+ docs/adr/0001-csv-import-idempotency.md  (仅当决策"重复日期覆盖策略"值得记录时)
```
```
/to-spec
```

**产出:** 一份 spec 发布到 issue tracker:

```
GitHub 模式:  https://github.com/<你>/<repo>/issues/42  ← 新 issue,带 ready-for-agent 标签
本地模式:     + .scratch/csv-import/spec.md
```
```
/to-tickets   (可选,大需求推荐加这一步)
```

**产出:** 拆分成多个可追踪的 ticket:

```
GitHub 模式:  多个 issue (每个 ticket 一个),标注 blocking 关系
本地模式:     .scratch/csv-import/
              ├─ spec.md
              └─ issues/
                 + 01-parse-csv-rows.md
                 + 02-validate-fields.md      (Blocked by: 01)
                 + 03-upsert-with-dedup.md    (Blocked by: 02)
                 + 04-summary-logger.md       (Blocked by: 01)
```

---

> ### ⚠️ 实施纪律:多 ticket 时,一次只 implement 一个
> 
> 拿到一堆 ticket 后,不要一次让 `/implement #42 #43 #44` 三个一起干。原因:
> 
> 1. **破坏 vertical slice** —— `/to-tickets` 拆 tracer bullet 就是为了让每个 ticket 可独立验证,合并做会毁掉这个属性
> 2. **上下文越界** —— AI 一次改多个 ticket,极容易"顺手"改到没让它动的地方,`/code-review` 也审不到跨界改动
> 3. **测试变糊** —— 全量测试挂了,分不清是哪个 ticket 造成的
> 4. **人类 review 变难** —— 一个 commit 塞 3 个功能,评审者(或未来的你)看得头大
> 
> **正确做法:按 `Blocked by:` 关系一个一个来。** 每个 ticket 都是独立的一次闭环:
> 
> ```
> Shift+Tab 进 Plan Mode → /implement <单个 ticket> → Approve → AI 实现 → commit
> ```
> 
> **执行顺序示例:** 拆出 `#01 #02 #03 #04` (见上方 blocking 关系):
> 
> ```
> #01(无依赖)   ─┬─→ #02(blocked by #01)
>                 │              │
>                 │              ↓
>                 │              #03(blocked by #02)
>                 └─→ #04(blocked by #01)
> ```
> 
> 顺序:**先 #01 → 完了再做 #02 或 #04(两者并列,任一先做都行)→ 最后 #03。** 带 `Blocked by:` 标记的 ticket,前置未完成绝不动手。
> 
> 远程模式看 GitHub issue 面板的 blocked 关系,本地模式看每个 `.md` 文件顶部的 `Blocked by:` 那行。

---

```
Shift+Tab   (切换到 Plan Mode)
/implement #01
按 01-parse-csv-rows 这个 ticket 实现。
```

**产出:** 真正动源代码:

```
+ src/importer/csv-parser.ts         (新增)
+ src/importer/csv-parser.test.ts    (新增)
~ src/importer/index.ts              (修改,挂载新解析器)
🔀 git commit: "feat(importer): CSV parser for K-line data"
```
```
/code-review
```

**产出:** 对话里输出评审报告(不写文件);发现的问题若需修复,会走另一个 commit。

---

> 📌 **过一遍完整链路后,你的仓库看起来是这样的(对比一开始的裸仓库):**
> 
> ```
> StockAnalyzer/
> ├─ CLAUDE.md                              ← setup 时创建
> ├─ CONTEXT.md                             ← grill-with-docs 创建
> ├─ docs/
> │  ├─ agents/{issue-tracker,triage-labels,domain}.md   ← setup
> │  └─ adr/0001-csv-import-idempotency.md              ← grill-with-docs(可选)
> ├─ src/importer/
> │  ├─ csv-parser.ts                       ← implement
> │  ├─ csv-parser.test.ts                  ← implement
> │  └─ index.ts (updated)                  ← implement
> └─ package.json
> ```

---

### 场景 2:量化核心计算逻辑(指标 / 策略 / 回测)

> ⚠️ **强制规范:** 涉及核心数值计算、资产结算等逻辑,必须强制走 `/tdd` 流程,先写 Failing Test 再写实现。

**推荐链路:** `/grill-with-docs` → `/to-spec` → `/tdd`

```
/grill-with-docs
实现 ATR(平均真实波幅)指标计算:输入 K 线序列,周期默认 14;严格遵循标准 ATR 算法,
兼容数据不足周期的异常场景;输出每条 K 线对应的 ATR 计算结果。
```

**产出:** 若沉淀了"ATR""真实波幅"等术语,更新 `CONTEXT.md` 。

```
/to-spec
```

**产出:** 一条 spec issue(含 seams 定义、user stories、testing decisions),用于指导后续 TDD 的测试点位。

```
/tdd
```

**产出模式与 `/implement` 根本不同——是"红→绿"多轮交替:**

```
─ 循环 1 ─ 覆盖"标准周期 14 的正常场景"
+ src/indicators/atr.test.ts   (只写这一个 failing test,先跑一次确认红)
+ src/indicators/atr.ts        (最小实现让这个 test 变绿)

─ 循环 2 ─ 覆盖"数据不足周期的边界"
~ src/indicators/atr.test.ts   (追加一个 failing test)
~ src/indicators/atr.ts        (改实现让新 test 也绿,不破坏旧 test)

─ 循环 3 ─ 覆盖"输入为空数组"
~ src/indicators/atr.test.ts   (再追加)
~ src/indicators/atr.ts        (再改)

...(继续直到所有 seam 覆盖)

🔀 git commit: 通常一个循环一次小 commit,或最终一次汇总 commit(由项目规范定)
```

**关键差异:**

- `/implement`:一次改到位、结束跑全量测试
- `/tdd`:**先写红色测试 → 只写让它变绿的最少代码 → 再写下一个红色测试** ——每一步都必须先看到测试失败(避免"测试写完就绿"的假象)

> 📌 **为什么核心业务红线要走 TDD:** 数值计算的 bug 常常"看起来对但边界错"(如漏掉 0/负数/精度)。红→绿循环强迫你 **先想清楚该 pass 什么、该 fail 什么** ——这个思考本身就是最好的边界防护。

---

### 场景 3：算法验证与原型探索

**推荐链路** ： `/grill-with-docs` → `/prototype`

```
/grill-with-docs
验证 5/20 日均线交叉策略：基于内存中的 K 线数据模拟回测，暂不接入数据库，统计总收益率与最大回撤。
```
```
/prototype
```

---

### 场景 4：复杂缺陷定位与排查（Bug Fixing）

```
/diagnosing-bugs
回测模块的最大回撤计算结果异常：人工核算为 12.5%，程序实际输出为 18.2%，请协助定位根本原因并补充回归测试用例。
```

---

### 场景 5：大型需求拆解与工单流转（团队协作）

```
/grill-with-docs
开发策略回测后台：包含新建策略、选择数据区间、执行回测、可视化收益指标展示及导出回测报告。
```
```
/to-spec
```
```
/to-tickets
```

> Agent 将自动读取 `docs/agents/issue-tracker.md` 并在 GitHub 创建对应的 Issues。

```
/triage
```

---

### 场景 6:跨需求平滑交接与上下文清理(防止对话污染)

开发完一个需求准备换到下一个需求时:

1. 在当前对话框输入 `/handoff`,导出当前需求的成果总结。
	- **产出位置:** OS 临时目录(**不污染工作区**)。Windows 通常在 `%TEMP%\` 下,macOS/Linux 在 `$TMPDIR/` 或 `/tmp/` 下。文件名类似 `handoff-<时间戳>.md`
		- **产出内容:** 已完成的部分、进行中的状态、下一步建议动作、推荐的 skills
2. **直接关闭当前对话窗口,新建一个干净的窗口** 。
3. 新窗口第一句话把 handoff 文件路径丢给 AI(比如"读一下 `C:\Users\<Your-Username>\AppData\Local\Temp\handoff-20260807-153000.md`,继续上次的工作")。
4. 新窗口中的 AI 会重新读取纯净的领域 Context + handoff 摘要,不会受到上一个需求繁杂调试对话的干扰。

---

## 7\. 运维管理与高级指令

### 查看本地已支持的 Agent 平台列表

```sh
npx skills@latest list-agents
```

### 查看技能的原始定义文件

每一个 Skill 的核心 Prompt 与执行逻辑均保存在其目录下的 `SKILL.md` 中：

```
# 示例路径
C:\Users\<Your-Username>\.agents\skills\grill-with-docs\SKILL.md
```

---

## 8\. 常见问题解答（FAQ）

### Q1：多个需求连续开发，CONTEXT.md 会不会变成“垃圾堆”污染下一个需求？

**不会。**

1. **职责分离** ： `CONTEXT.md` 只录入业务词汇与规则约束，不存代码和临时接口细节。
2. **提纯更新** ： `/grill-with-docs` 会自动更新修改旧词条，而不是盲目追加。
3. **物理隔离** ：大型项目可配置 `CONTEXT-MAP.md` 分子系统管理（可直接指示 AI 自动拆分）。
4. **最佳习惯** ：换新需求时务必 **新建 AI 对话窗口** ，彻底清除对话历史 Memory。

### Q2：Multi-Context 架构需要我自己创建目录和文件吗？

不需要。你可以直接发送指令让 AI 帮建立，或在运行 `/improve-codebase-architecture` 时根据 AI 提示确认自动建构。

### Q3：为什么执行了 /setup-matt-pocock-skills 之后没看到 CONTEXT.md 和 docs/adr/？

这是正常现象。最新版架构采用了 **按需懒加载（Lazy Load）** 机制。只有在第一次调用 `/grill-with-docs` 形成业务收炼时才会建立。

### Q4：为什么输入斜杠命令（如 /grill-with-docs）AI 没有触发技能逻辑？

1. 确认技能是否成功安装:执行 `npx skills@latest list-agents` 或直接查看 `~/.agents/skills/` 是否有对应目录。
2. 检查客户端是否识别到 skills:在 Claude Code 里敲 `/`,应能看到候选命令列表;敲不出来说明客户端没加载到。
3. 特定命令(如 `/grill-with-docs` 、 `/to-tickets`)依赖 `docs/agents/` 下的配置,若未跑过 `/setup-matt-pocock-skills`,可能行为异常。 **注意:普通命令(如 `/grilling` 、 `/tdd`)不依赖 `docs/agents/`,可以在裸仓库直接用。**
4. 尝试重启 VSCode 或重新加载 Claude Code 插件。

### Q5：在 Codex CLI 等非 VSCode 界面下如何正确使用？

在 Codex CLI 界面中,斜杠语法与 Claude Code 不同——通常需要 **移除命令开头的斜杠 `/`**,直接用 `setup-matt-pocock-skills` 或 `tdd` 唤起。

> 📌 不同客户端的实际调用语法可能微调,以各客户端最新文档为准;若调用失败可先跑一次 `npx skills@latest add mattpocock/skills -a codex` 确认 Codex 侧的 skills 已挂载。

### Q6：Symlink（软链接）和 Copy 模式应该怎么选择？

首选 **Symlink 模式** 。多个 Agent 共享同一套技能库，且可以通过 `npx skills update` 一键更新全局生效。

---

## 9\. 团队开发规范约定

1. **一需求一窗口** ：每个需求开发完毕并 Commit 后，必须开启新的 AI 会话窗口，避免历史对话 Token 污染。
2. **词汇统一原则** ：AI 产生的任何测试名、函数名与 Issue 标题，必须严格遵守 `CONTEXT.md` 定义的领域术语。
3. **核心业务红线** ：涉及核心数值计算、交易逻辑、资金安全的代码，必须采用 `/tdd` 模式开发，不允许直接使用 `/implement` 跳过测试。
4. **初始化规范** ：任何新项目或新拉取的仓库，在接入 AI 协同前必须运行一次 `/setup-matt-pocock-skills` 。
5. **冲突显性化** ：如果新的实现违反了已有的 ADR 决策，Agent 必须显式提示（如 `Contradicts ADR-0007...`），严禁静默覆盖。
6. **Plan Mode 优先(Claude Code 使用者)**:执行 `/implement` 、 `/tdd` 等会改动代码的技能前,先按 `Shift+Tab` 切到 **Plan Mode** ——AI 只读、不改,先给出改动计划,人工 review 通过再执行。 **这是防止"AI 改飞"的最关键护栏,尤其在生产代码/核心业务模块必须开启。**