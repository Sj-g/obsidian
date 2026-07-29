---
title: "如何为 Obsidian 配置 AI Agent？9 个必备 Skill 详解与安装指南 | 杰森的效率工坊"
source: "https://jasonai.me/blog/obsidian-ai-agent-skills-configuration-guide/"
author:
  - "[[Jason]]"
published: 2026-04-03
created: 2026-07-29
description: "Obsidian 知识库使用 AI Agent 必须选择合适的 Skill，规避过时的 OpenClaw 与 JSON-Canvas，改用 Axton 重构的三大画图 Skill。  Tutor 与 Scholar 工作流分别实现 PDF 自动双链转换和学术科研笔记，配置代码已整理至 Markdown 文档可在主页下载。"
tags:
  - "clippings"
---
## Obsidian 必装 Skills

| 作者                                                                       | Skill名称                 | 功能描述                                                                                                                | 推荐度 |
| ------------------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------- | --- |
| **Obsidian CEO @kepano**   GitHub: `kepano/obsidian-skills`              | defuddle                | 网页内容清洗工具，专门用来把杂乱的网页转换成纯净的 Markdown 格式，通过剔除广告和导航栏来帮你节省 AI 调用时的 Token 消耗。                                             | ✅   |
|                                                                          | obsidian-cli            | 让 AI Agent 能够直接调用 Obsidian 官方的命令行工具，从而实现对笔记、任务、属性的增删改查，以及对插件开发环境的调试与管理。                                             | ✅   |
|                                                                          | obsidian-bases          | 让 AI 能够创建和维护.base 格式的配置文件，从而在 Obsidian 里生成类似 Notion 数据库的动态视图，实现对笔记的过滤、计算和结构化展示。                                     | ✅   |
|                                                                          | obsidian-markdown       | 让 AI 能够编写和编辑符合 Obsidian 官方规范的增强版 Markdown 文档，实现双向链接、内容嵌入、提示框以及结构化属性的深度集成。                                           | ⚠️  |
|                                                                          | json-canvas             | 让 AI 能够创建和编辑 Obsidian 的.canvas 白板文件，通过 JSON 结构实现节点（文本、文件、链接、组）的布局以及它们之间的连线逻辑。                                       | ❌   |
|                                                                          |                         |                                                                                                                     |     |
| **Axton，著名博主@回到Axton**   GitHub: `axtonliu/axton-obsidian-visual-skills` | obsidian-canvas-creator | 加强版的 json canvas skill，解决了节点重叠和空间分布不均的问题。                                                                           | ✅   |
|                                                                          | mermaid-visualizer      | 将文本逻辑转化为专业的 Mermaid 架构图或流程图，并内置了针对 Obsidian 渲染引擎的语法纠错机制。                                                            | ✅   |
|                                                                          | excalidraw-diagram      | 将文本逻辑转化为手绘风格的 Excalidraw 图表                                                                                         | ✅   |
|                                                                          |                         |                                                                                                                     |     |
| **OpenClaw官方GitHub**   GitHub: `openclaw/openclaw/skills/obsidian`       | obsidian-skill          | 直接操作文件系统，也就是文件I/O，非常消耗Token，在官方已经发布Obsidian-cli的情况下，没有理由继续使用这个过时的方式。                                                | ❌   |
|                                                                          |                         |                                                                                                                     |     |
| **Choi Wontak**   GitHub: `RoundTable02/tutor-skills`                    | tutor-skills            | 两个 Skill (tutor-setup 和 tutor) ，构成了一个“输入-内化-检测”的完整闭环：将文档或代码库一键转化为结构化的 Obsidian 知识库，之后通过无提示的交互式测验不断暴露出你的知识盲区并记录学习轨迹。 | ✅   |
|                                                                          |                         |                                                                                                                     |     |
| **EESJGong**   GitHub: `EESJGong/scholar-skill`                          | scholar-skill           | 基于 OpenClaw 框架的学术研究skill，通过L1-L3的分级阅读策略在后台长时间静默解析论文，并自动将结构化笔记、核心记忆与知识冲突报告写入你的本地 Obsidian 知识库中。                      | ✅   |

## Obsidian CEO 发布的 Skills

### defuddle

#### Skill功能

Defuddle 主要用来抓取网页里的核心正文。它会自动删掉导航条、侧边栏和广告等干扰元素，只留下干净的 Markdown 内容。这个功能让 AI 在处理长文章、在线文档或博客时，不仅能读得更准，还能大幅减少不必要的字符开销。最近一次更新中已经支持YouTube视频链接，它获取YouTube视频字幕的方式是调用YouTube官方API，而不是我们之前熟知的 `yt-dlp` 组件。

#### 所需依赖

1. Node.js 运行环境。
2. 全局安装的 defuddle 包，安装命令是 `npm install -g defuddle` 。

#### 触发条件和使用方法

当你给 Agent 发送一个网页链接（URL），并要求它阅读、分析或总结里面的内容时，就会触发这个 Skill。

提示词样例：

```text
提取这个网页的正文，转成干净的 Markdown 格式：[URL]。
```

#### 注意事项

这个工具对标准 HTML 网页（如新闻、博客、官方文档）效果最好，但如果网页需要登录或者是纯动态渲染的单页应用，抓取效果可能受限。

### obsidian-cli

**功能开启与基础配置**

1. 确认系统环境，保证 Obsidian 客户端版本在 1.12 以上。
2. 进入 Obsidian 界面，打开“设置 -> 常规 (General)”。
3. 找到“命令行界面 (Command line interface)”开关并打开。
4. 在弹出的窗口中确认注册到系统 Path。
5. 保证 Obsidian 客户端处于运行状态（如果在未运行状态下执行命令，系统会自动启动客户端）。

验证配置成功的方法是打开操作系统的终端工具（Mac 使用 Terminal，Windows 使用 PowerShell），输入下面这个基础命令：

```bash
obsidian daily
```

如果配置正确，Obsidian 会直接自动应用日记模版并在界面中生成今天的日记文件。

### obsidian-bases

#### Skill 功能

这个 Skill 允许 AI 通过编写 YAML 格式的 `.base` 文件来创建Obsidian bases数据库。它支持非常强大的公式系统，可以读取笔记的 Frontmatter 属性或文件元数据（如创建时间、字数等），并进行条件判断、日期运算和字符串处理。

#### 所需依赖

- **Maps 插件** ：如果需要使用这个 Skill 里的 map（地图）视图，必须额外安装名为 Maps 的社区插件。

#### 触发条件和使用方法

样例提示词：

```text
写一个 .base 文件来管理我的项目笔记，要求筛选出所有带 #project 标签的文件，用表格显示项目名称、截止日期和剩余天数。
```

**视图限制** ：目前只支持 table（表格）、cards（卡片）、list（列表）和 map（地图）这四种类型。

### obsidian-markdown

#### 使用方法

样例提示词：

```text
根据这段会议记录生成一份 Obsidian 笔记。要求在顶部包含日期和参会人的属性，使用双向链接关联到“项目 A”笔记，并把关键决策点用important类型的提示框标注。
```

**扩展** ：因为skill是对格式的语法和约束，所以可以将个人obsidian格式偏好加入到skill中，以保证Agent写出的知识笔记符合你的要求。

### json-canvas

#### 使用方法

样例提示词：

```text
创建一个名为“AI 学习路径”的 json canvas文件。中心是一个文本节点，连向三个文件节点，分别是“模型基础”、“提示词工程”和“智能体实战”。
```

### obsidian-canvas-creator

#### Skill 功能

内置了径向布局（MindMap）和自由排版（Freeform）算法，能够自动计算节点坐标、处理连接线路径、并根据文本长度动态调整节点尺寸，从而生成整齐、美观且逻辑清晰的视觉架构。

#### 使用方法

样例提示词：

```text
把这markdown知识笔记转换成一个mindmap格式的 Obsidian Canvas。
```

#### 与 json-canvas skill 的对比

- **侧重点不同** ： `json-canvas` 主要关注底层的 JSON 语法正确性和属性定义；而 `obsidian-canvas-creator` 侧重于高层的排版策略和空间坐标计算。
- **自动化程度不同** ：使用 `json-canvas` 时，AI 可能只是机械地摆放节点；使用本 Skill 时，AI 会根据 `layout-algorithms.md` 计算每个节点的 X/Y 坐标，确保节点不重叠且间距符合视觉审美。
- **结构化逻辑** ：本 Skill 引入了特定的布局模式（如 MindMap 模式和 FreeForm模式），能自动处理父子层级关系，而不仅仅是单一的节点创建。

### mermaid-visualizer

#### 使用方法

样例提示词：

```text
根据这段关于软件开发生命周期的描述生成一个横向的 Mermaid 流程图，要求包含不同的子图来区分开发环境和测试环境。
```

### excalidraw-diagram

#### 所需依赖

- **Excalidraw 插件** ：必须安装并启用 Excalidraw 插件。

#### 触发条件和使用方法

样例提示词：

```text
用 Excalidraw 画一个 AI 智能体的工作流动画图，按照感知、思考、行动的顺序设置动画步骤，并保存为 .excalidraw 文件。
```

### tutor-skills

#### Skill 功能

`tutor-setup` 能将任何本地文档或源代码工程自动解析，并生成带有双链、MOC 和复习题的独立 Obsidian 学习金库 (StudyVault)；而 `tutor` 则读取知识库的进度数据，在终端内为你生成互动式测验，追踪并攻克你的知识薄弱点。

#### 所需依赖

- **基础环境** ：智能体工具如 Claude Code, OpenCode。

#### 使用方法

- **使用方法** ：在特定工作目录输入命令 `/tutor-setup` 触发构建，或在已有 StudyVault 的目录下输入 `/tutor` 触发复习。

#### Skill 的特殊机制

- **模式自动侦测** ：无需手动指定，Skill 会自动扫描当前工作目录，若发现 `package.json` 或 `pom.xml` 等工程文件会自动进入“代码库模式”；若只有 PDF/纯文本，则进入“文档模式 ”。

#### 注意事项

- **Token 消耗风险** ：尽管禁止了 PDF 图像读取，但“代码库模式”会递归读取大量源文件并进行架构溯源（Phase C1-C9 循环），在短时间内消耗大量 Token 额度。

### scholar-skill

#### Skill 功能

`scholar-skill` 是一个深度的个人知识管理与文献解构工作流。它通过分级标准（L1分发/L2标准阅读/L3深度解构），将原始论文（PDF/ArXiv）转化为 Obsidian 中的双链卡片、MOC（内容地图）以及系统性的反思报告。它还能记录你阅读过程中的误区并提取可复用的研究方法论。

#### 所需依赖

要运行此系统，你需要配置一套相对重型的底层环境：

- **基础环境** ：本地 Python 环境与预先安装好的 Obsidian 客户端（及配置好的 Vault 文件夹）。
- **核心框架** ：安装配置好的 OpenClaw 智能体框架。
- **依赖 Skill (通过 ClawHub 安装)** ：
	- `obsidian-direct` （必须）：用于绕过官方限制，直接通过 Python 强行读写本地 `.md` 文件。
		- `arxiv-watcher` （必须）：用于通过 ArXiv API 抓取文献资源。
		- `durable-task-runner` （核心必须）：用于支持 L3 级别长时间挂机任务的调度与断点续传。
		- 增强依赖（可选）： `tavily` （联网抓取）、 `pdf` （文本解析）、 `academic-research-hub` 。

#### 触发条件与使用方法

- **触发条件** ：当意图匹配到“阅读论文”、“L1/L2/L3阅读”、“知识内化”或“文献笔记”时自动触发工作流。
- **使用方法（提示词样例）** ：
```text
获取这篇文献 ArXiv:2407.19354 并进行处理。
先做 L1 快速评估，如果判定为 P0 优先级，则请在后台直接启动 L3 深度阅读。
完成后将知识树更新推送到我的 Obsidian 对应目录。
```

#### Skill 的特殊机制

- **超长周期任务编排** ：由于大模型无法一次性吃透几十页附带复杂公式的论文，L3级深度阅读被设计为长达 2.5 小时的异步挂机任务。底层深度依赖 OpenClaw 的 `durable-task-runner` 来处理多次 LLM 推演循环、API 限流等待以及崩溃恢复。
- **周期性反思机制** ：内置时间触发器逻辑，强制在周末或月末对“临时存储的知识”进行 L2/L3 反思，生成知识体系演进报告。
- **人类确认防呆机制** ：当 AI 发现新论文推翻了你旧笔记的结论时，不会直接覆写旧笔记，而是生成一份确认单放进 `0-Inbox` 文件夹，等待人类审核确认（Human in the loop）。

#### 注意事项与风险预警

- **财务毁灭/算力黑洞风险** ：长达 2.5 小时的 L3 循环和高频的历史知识检索（RAG）会消耗极其恐怖的 Token。如果后端挂载的是商用前沿模型（如 Claude 3.5 Sonnet 或 GPT-4o），单篇深读可能带来高昂的 API 账单。
- **数据覆写与坏档风险** ：底层的 `obsidian-direct` 使用的是民间 Python 暴力文件 I/O 脚本，而非 Obsidian 官方 CLI 通信。在文件多端同步（如 iCloud/Obsidian Sync）期间，极易引发文件冲突、内容丢失或双链索引错误。强烈建议在独立测试库中运行，并开启 Git 快照。

---

## 核心插件

- **claudian**: Obsidian 第三方插件（暂未上架官方市场），适配 Claude Code。GitHub Repo: `YishenTu/claudian`
- **obsidian-agent-client**: 第三方插件（暂未上架官方市场），适配主流智能体：Claude Code, Codex, Gemini CLI, OpenCode, Qwen Code。GitHub Repo: `RAIT-09/obsidian-agent-client`

### 1\. 安装方式

#### 方案 A：通过 BRAT 安装 (推荐)

这是保持插件自动更新的最佳方式，适合尚未上架市场的 Beta 插件。

1. **安装 BRAT**: 在 Obsidian 插件市场搜索并安装 BRAT。
2. **添加仓库**: 打开 设置 -> BRAT -> Add Beta plugin，输入仓库地址：YishenTu/claudian。
3. **启用**: 点击 Add Plugin 等待下载完成后，在“第三方插件”列表中开启 **Claudian** 。

#### 方案 B：手动加载

若网络环境无法直接连接 GitHub 仓库，可采用此法。

1. **获取文件**: 访问 GitHub 仓库 Releases 页面，下载 main.js、manifest.json、styles.css。
2. **创建路径**: 进入仓库目录.obsidian/plugins/，新建文件夹 claudian。
3. **放置文件**: 将下载的三个文件放入该文件夹。
4. **启用**: 重启 Obsidian 或在插件设置页刷新，手动开启。

### 2\. 插件设置

#### claudian设置

1. 打开 claudian 设置页。
2. **基础设置**: 设置 `User Name` (如 Jason)。
3. **自定义AI模型**: 使用兼容Anthropic接口的模型（如智谱GLM或DeepSeek）来替换Claude模型。
```bash
ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/anthropic
ANTHROPIC_API_KEY=你的智谱api key
ANTHROPIC_DEFAULT_OPUS_MODEL=GLM-5
```
4. **连通性验证**:
- `Ctrl/Cmd + P` 调出命令面板 -> 输入 `claudian` -> 选择 `Open chat view` 。
- 发送“你好”，若回复正常则配置成功。

#### obsidian-agent-client设置

以 OpenCode为例：

1. 打开 agent-client 设置页。在 Custom Agent 中，点击 `Add Custom Agent` 按钮。设置 Agent ID 和 Display Name 为 OpenCode。
2. 在命令行中输入 `where opencode` 查看 OpenCode 安装路径，把路径填入Path。
3. Arguments 填如下信息，注意第三行是你的Obsidian Vault路径：
```bash
acp
--cwd
D:\Obsidian Vault\MyObVault
```
4. **连通性验证**:
- 在Agent Client的AI对话框中，发送“你好”，若回复正常则配置成功。