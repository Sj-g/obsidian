---
title: "Karpathy"
type: entity
tags: [实体, 人物, AI研究者, LLM-Wiki]
sources: [raw/01-articles/llm-wiki.md]
last_updated: 2026-07-29
---

## 定义
**Andrej Karpathy** 是知名的 AI 研究者与教育者，本知识库奠基方法论 **[[LLM Wiki]]** 的提出者。他在 GitHub Gist（"llm-wiki"）中系统阐述了"用 LLM 构建持久、互链的个人知识库"这一模式，该文档也是本 vault 的 CLAUDE.md 所引用的核心规范来源。

## 关键信息
- **核心贡献（本库语境）**：提出 [[LLM Wiki]] 方法论——以三层架构（raw / wiki / schema）与三大操作（ingest / query / lint）替代 [[RAG]] 的"每次重推导"范式。
- **关键洞见**：维护知识库的瓶颈不在阅读思考，而在繁琐的 bookkeeping；LLM 使维护成本趋近于零，从而让 [[Memex]] 的愿景可持续。
- **工作流主张**：人类负责筛选素材与提问，LLM 负责所有维护工作。

## 关联连接
- [[LLM Wiki]] — 提出的核心方法论
- [[摘要-llm-wiki]] — 其 Gist 原文摘要
- [[RAG]] — 其方法论所对比的范式
- [[Memex]] — 其思想引用的先驱
