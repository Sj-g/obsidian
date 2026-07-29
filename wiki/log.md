# Wiki Operation Log

## [2026-07-29] ingest | 摄入 raw/01-articles 下 5 个文件，完成知识库冷启动
- **变更**:
  - 新增 5 个来源摘要：[[摘要-438c-产品文档体系]]、[[摘要-需求拆分过程规程]]、[[摘要-36本书书单]]、[[摘要-每日站会素材]]、[[摘要-待看名单]]
  - 新增 1 个实体：[[GJB438C]]（作为文档体系与需求规程的枢纽节点）
  - 新增 10 个概念：[[需求分析与拆分]]、[[纵向拆分]]、[[INVEST准则]]、[[影响地图]]、[[需求优先级评估]]、[[MVP]]、[[复利效应]]、[[多元思维模型]]、[[系统思考]]、[[每日站会]]
  - 重写 [[index]]（原为空），并将既有 synthesis `5c-prompt-markdown-note-taking` 纳入注册
  - 已将 5 个源文件归档至 raw/09-archive/
- **冲突**: 无知识冲突（本次为冷启动，wiki 此前为空）
- **遗留问题（待 /lint 处理）**: 既有页面 `5c-prompt-markdown-note-taking` 存在一批死链——其中 `5C_Framework`、`Prompt_Engineering`、`摘要-5c-prompt-contracts-paper` 为真实的未建页面；`番茄工作法`、`注意力经济`、`双向链接`、`页面名称` 则是该页提示词模板/示例里的占位文字（非真正知识链接，会被 lint 误报）。该页此前为孤儿，现已通过 index 注册获得入链。建议后续补充对应 raw 素材后 ingest，或运行 `/lint` 集中清理。
- **健康检查**: 本次新增的 16 个页面（5 sources + 1 entity + 10 concepts）经扫描，死链 0、孤岛 0。

## [2026-07-29] query | 解答"什么是 438C"
- **输出**: 引用 [[GJB438C]]、[[摘要-438c-产品文档体系]]、[[摘要-需求拆分过程规程]]、[[需求分析与拆分]]；即时回答（待用户确认是否固化为 synthesis）

## [2026-07-29] lint | 清除全部死链（指向不存在文件的链接）
- **变更**:
  - index.md：移除已丢失的 synthesis 条目（`5c-prompt-markdown-note-taking`），Syntheses 分类置空
  - log.md：将历史日志中 8 处指向不存在文件的内联链接去链接化为纯文本（保留文字记录，仅消除无效 wikilink）
- **说明**: `wiki/syntheses/5c-prompt-markdown-note-taking.md` 文件已丢失（非本会话操作所致，疑似 Obsidian 同步/外部变更）；用户指示"没有的都清除掉"，选择清除而非重建
- **结果**: 死链 8 → 0；16 个知识页面维持死链 0、孤岛 0、无冲突

## [2026-07-29] ingest | 摄入 Karpathy LLM Wiki 方法论原文（奠基文档）
- **变更**:
  - 新增 1 个来源摘要：[[摘要-llm-wiki]]
  - 新增 2 个实体：[[Karpathy]]、[[Obsidian]]
  - 新增 3 个概念：[[LLM Wiki]]、[[RAG]]、[[Memex]]
  - 更新 [[index]]（LLM Wiki 系列置于各分类首位，体现奠基地位）
  - 已将源文件归档至 raw/09-archive/
- **冲突**: 无
- **意义**: 本资料是 CLAUDE.md 引用的 Karpathy LLM-Wiki gist 原文，定义了本库的 ingest/query/lint 三大操作与三层架构。[[LLM Wiki]] 概念页已标注本库的自指关系——知识库完成了对自身运作模式的元认知。

## [2026-07-29] query | 解答"每日站会"与"Marp"
- **输出**: 引用 [[每日站会]]、[[Obsidian]]、[[摘要-llm-wiki]]；Marp 在 wiki 无独立页面，补充通用知识并建议补建概念页；即时回答未保存
