---
title: "Obsidian"
type: entity
tags: [实体, 工具, 知识管理, Markdown]
sources: [raw/01-articles/llm-wiki.md]
last_updated: 2026-07-29
---

## 定义
**Obsidian** 是一款基于本地 Markdown 文件的知识管理软件，是 [[LLM Wiki]] 工作流中的核心工具。在 Karpathy 的比喻中：

> Obsidian 是 IDE；LLM 是程序员；wiki 是代码库。

它通过双向链接、图谱视图与丰富的插件生态，承担 LLM Wiki 的"浏览与导航层"——LLM 负责写入，用户在 Obsidian 中实时浏览更新、追踪链接、检视结构。

## 关键信息
**在 LLM Wiki 工作流中的角色（来自 [[摘要-llm-wiki]]）：**
- 实时浏览 LLM 的编辑结果：点击链接、查看图谱、阅读更新页。
- **Graph view（图谱视图）**：观察 wiki 形态的最佳方式——看清哪些页面是枢纽、哪些是孤儿。
- 作为 git 仓库的 Markdown 文件集，天然获得版本历史、分支与协作。

**相关插件/扩展：**
- **Dataview**：对页面 frontmatter（YAML）跑查询，生成动态表格/列表。
- **Marp**：基于 Markdown 的幻灯片格式插件，可从 wiki 内容直接生成演示。
- **Obsidian Web Clipper**：浏览器扩展，把网页文章转为 Markdown，快速入 raw 库。
- 图片本地化：设置固定附件目录（如 `raw/assets/`）+ 下载快捷键，避免 URL 失效。

## 关联连接
- [[LLM Wiki]] — 其作为 IDE 参与的工作流
- [[Memex]] — 体现其关联存储理念的工具化
- [[摘要-llm-wiki]] — 角色论述来源
