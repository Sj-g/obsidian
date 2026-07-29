---
title: "LLM Wiki"
type: concept
tags: [概念, 知识管理, 方法论, LLM, 第二大脑]
sources: [raw/01-articles/llm-wiki.md]
last_updated: 2026-07-29
---

## 定义
**LLM Wiki** 是 [[Karpathy]] 提出的一种个人知识库构建方法论：让大语言模型（LLM）**增量构建并维护一个持久的、相互链接的 Markdown 知识库**，使其夹在用户与原始资料之间。与 [[RAG]]"每次查询从头检索"不同，LLM Wiki 把知识**编译一次、持续保鲜**——wiki 是一个"持续复利"的产物（persistent, compounding artifact）。

> 人类负责筛选素材、探索方向、提出好问题；LLM 负责所有繁琐的维护——总结、交叉引用、归档、记账。Obsidian 是 IDE，LLM 是程序员，wiki 是代码库。

## 关键信息
**三层架构：**

| 层 | 角色 | 对应本库 |
|----|------|----------|
| Raw sources | 不可变源文件，唯一真相 | `raw/`（只读） |
| The wiki | LLM 拥有的 Markdown 目录 | `wiki/` |
| The schema | 结构/规范/工作流配置 | `CLAUDE.md` |

**三大操作：**
- **Ingest**：新源 → 提炼 → 写摘要 + 更新 index/实体/概念页 + 追加 log（单源可触及 10-15 页）。
- **Query**：检索 → 阅读 → 带引用综合；好答案回填为新页，让探索也复利。
- **Lint**：定期查矛盾、过时声明、孤儿页、缺失概念页与交叉引用。

**两个导航文件：** `index.md`（内容导向目录，query 入口）、`log.md`（时间导向 append-only 日志，统一前缀便于 `grep` 解析）。

**适用场景：** 个人成长追踪、深度研究、读书伴侣（类比 Tolkien Gateway 粉丝 wiki）、团队内部 wiki、竞争分析、课程笔记等任何"随时间积累、希望有序而非散乱"的场景。

## 自指：本知识库即此模式的实例
**你现在浏览的这个 Obsidian vault，就是 LLM Wiki 模式的实例化。** CLAUDE.md 是其 schema 层；`raw/`→`wiki/` 的编译流水线即 ingest 操作；[[GJB438C]]、[[需求分析与拆分]] 等所有页面都是 ingest 产出的实体/概念节点。本页本身则是该模式对自身的元认知。

## 关联连接
- [[摘要-llm-wiki]] — 方法论原文摘要
- [[Karpathy]] — 提出者
- [[RAG]] — 对比范式（每次重推导 vs 编译一次）
- [[Memex]] — 历史先驱（Vannevar Bush, 1945）
- [[Obsidian]] — 工作流 IDE
- [[GJB438C]] / [[需求分析与拆分]] — 本库中的实例节点
