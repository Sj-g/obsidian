---
title: "llm-wiki"
source: "https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f"
author:
  - "[[262588213843476]]"
published:
created: 2026-07-29
description: "llm-wiki. GitHub Gist: instantly share code, notes, and snippets."
tags:
  - "clippings"
---
## LLM Wiki LLM 维基

A pattern for building personal knowledge bases using LLMs.  
一种利用大语言模型构建个人知识库的方法论。

This is an idea file, it is designed to be copy pasted to your own LLM Agent (e.g. OpenAI Codex, Claude Code, OpenCode / Pi, or etc.). Its goal is to communicate the high level idea, but your agent will build out the specifics in collaboration with you.  
这是一个概念文件，旨在被复制粘贴到您自己的语言模型代理中（例如 OpenAI Codex、Claude Code、OpenCode / Pi 等）。它的目的是传达高层次的概念，而您的代理将在与您合作的基础上进一步细化这些概念。

## The core idea 核心思想

Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation. Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.  
大多数人使用大语言模型和文档时的体验类似于 RAG 模式：用户上传一系列文件，大语言模型在查询时检索出相关部分内容，然后生成答案。这种方式确实可行，但大语言模型每次提问都需要从头开始重新获取知识，没有知识积累的过程。如果提出一个需要综合五份文档的复杂问题，大语言模型每次都需要找到并整合相关信息。没有任何知识被积累起来。NotebookLM、ChatGPT 的文件上传功能，以及大多数 RAG 系统都是采用这种运作方式。

The idea here is different. Instead of just retrieving from raw documents at query time, the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of markdown files that sits between you and the raw sources. When you add a new source, the LLM doesn't just index it for later retrieval. It reads it, extracts the key information, and integrates it into the existing wiki — updating entity pages, revising topic summaries, noting where new data contradicts old claims, strengthening or challenging the evolving synthesis. The knowledge is compiled once and then *kept current*, not re-derived on every query.  
这里的思路有所不同。与在查询时直接从原始文档中提取信息不同，大语言模型会逐步构建并维护一个持久的“维基”数据库——一个结构化的、相互链接的 Markdown 文件集合。当你添加一个新的数据源时，大语言模型不仅会将其索引以便后续检索，还会读取该数据、提取关键信息，并将其整合到现有的维基数据库中。这样就能更新实体页面、修正主题摘要，同时记录新数据与旧数据之间的冲突点，从而加强或挑战现有的信息整合方式。这些知识只需编译一次即可，无需在每次查询时重新生成。

This is the key difference: **the wiki is a persistent, compounding artifact.** The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.  
这就是关键的区别：维基是一个持续发展的、不断充实的数据库。所有的交叉引用都已经存在了；矛盾之处也已经被标记出来。所有的信息都已经体现在了综合结果中。每当你添加一个新的来源、解答一个新的问题，维基就会变得更加丰富起来。

You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping that makes a knowledge base actually useful over time. In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.  
你很少自己编写维基页面的内容——所有内容都是由大型语言模型来撰写和维护的。你的职责是寻找合适的素材、进行探索，并提出正确的问题。而大型语言模型则负责所有的琐碎工作，比如总结信息、建立交叉引用、整理文件、记录数据等等。这些工作使得知识库能够随着时间的推移而变得有用。实际上，我现在同时打开着大型语言模型代理和 Obsidian 软件。大型语言模型会根据我们的对话进行编辑，而我则实时浏览这些结果：点击链接、查看图表、阅读更新的页面内容。Obsidian 就像是开发环境；大型语言模型则是编程工具；而维基页面则代表了整个代码库。

This can apply to a lot of different contexts. A few examples:  
这种情况可以适用于许多不同的情境。以下是一些例子：

- **Personal**: tracking your own goals, health, psychology, self-improvement — filing journal entries, articles, podcast notes, and building up a structured picture of yourself over time.  
	个人方面：跟踪自己的目标、健康状况、心理状态以及自我提升情况——记录日常笔记、文章、播客笔记等，从而构建出随时间变化的自我形象。
- **Research**: going deep on a topic over weeks or months — reading papers, articles, reports, and incrementally building a comprehensive wiki with an evolving thesis.  
	研究：深入探究某个主题，可能需要数周或数月的时间——阅读相关论文、文章和报告，逐步构建出一个包含丰富信息的综合知识库，并不断完善其论点。
- **Reading a book**: filing each chapter as you go, building out pages for characters, themes, plot threads, and how they connect. By the end you have a rich companion wiki. Think of fan wikis like [Tolkien Gateway](https://tolkiengateway.net/wiki/Main_Page) — thousands of interlinked pages covering characters, places, events, languages, built by a community of volunteers over years. You could build something like that personally as you read, with the LLM doing all the cross-referencing and maintenance.  
	阅读一本书的过程就像整理每一章的内容：为角色、主题、情节线索以及它们之间的联系建立详细的页面记录。最终，你会得到一个内容丰富的配套维基百科。可以把粉丝维基类比成《托尔金之门》——成千上万的相互关联的页面，涵盖了角色、地点、事件、语言等细节，这些页面是由一群志愿者经过多年时间精心构建而成的。你在阅读时也可以自己创建类似的工具，让大语言模型负责所有的交叉引用和维护工作。
- **Business/team**: an internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls. Possibly with humans in the loop reviewing updates. The wiki stays current because the LLM does the maintenance that no one on the team wants to do.  
	业务/团队：这是一个由大型语言模型维护的内部维基页面，其数据来源包括 Slack 聊天记录、会议记录、项目文档以及客户通话记录。可能还会有人参与进来，对更新内容进行审核。由于大型语言模型负责维护该维基页面的内容，所以它能够保持最新的状态，而团队中的其他人则不愿意承担这项任务。
- **Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives** — anything where you're accumulating knowledge over time and want it organized rather than scattered.  
	竞争分析、尽职调查、旅行计划制定、课程笔记、兴趣爱好深入探讨——任何需要随时间积累知识并且希望将这些知识进行整理而不是保持零散状态的情况。

## Architecture 建筑学

There are three layers:  
它包含三层结构：

**Raw sources** — your curated collection of source documents. Articles, papers, images, data files. These are immutable — the LLM reads from them but never modifies them. This is your source of truth.  
原始资料——这是您精心整理的一系列源文件集合。包括文章、论文、图片、数据文件等。这些资料是不可篡改的——大语言模型可以读取这些资料，但不会对其进行任何修改。这里是您的权威信息来源。

**The wiki** — a directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, an overview, a synthesis. The LLM owns this layer entirely. It creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent. You read it; the LLM writes it.  
这个维基页面是一个由大语言模型生成的 Markdown 文件目录。其中包含摘要、实体页面、概念页面、对比分析、概述以及综合介绍等内容。整个页面由大语言模型负责创建和维护：模型会生成页面内容，在获得新数据时进行更新，同时维护着所有交叉引用关系，确保一切信息的统一性。页面内容由大语言模型生成，而实际内容的呈现则由模型完成。

**The schema** — a document (e.g. CLAUDE.md for Claude Code or AGENTS.md for Codex) that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow when ingesting sources, answering questions, or maintaining the wiki. This is the key configuration file — it's what makes the LLM a disciplined wiki maintainer rather than a generic chatbot. You and the LLM co-evolve this over time as you figure out what works for your domain.  
该文件实际上是一个文档，例如用于管理 Claude 代码的 CLAUDE.md 文件，或者用于 Codex 的 AGENTS.md 文件。它向 LLM 说明了 wiki 的结构如何、有哪些规范需要遵守，以及在获取信息、回答问题或维护 wiki 时应该遵循哪些工作流程。这是非常重要的配置文件——它使得 LLM 能够成为一个有纪律的 wiki 维护者，而不仅仅是普通的聊天机器人。随着时间的推移，你和 LLM 会共同完善这个配置文件，直到找到适合你所在领域的最佳方式。

## Operations 运营工作

**Ingest.** You drop a new source into the raw collection and tell the LLM to process it. An example flow: the LLM reads the source, discusses key takeaways with you, writes a summary page in the wiki, updates the index, updates relevant entity and concept pages across the wiki, and appends an entry to the log. A single source might touch 10-15 wiki pages. Personally I prefer to ingest sources one at a time and stay involved — I read the summaries, check the updates, and guide the LLM on what to emphasize. But you could also batch-ingest many sources at once with less supervision. It's up to you to develop the workflow that fits your style and document it in the schema for future sessions.  
输入。您可以将新的来源添加到原始集合中，然后指示大语言模型进行处理。举个例子：大语言模型会阅读该来源的内容，与您讨论关键要点，然后在维基页面上撰写摘要，更新索引，并在维基页面上的相关实体和概念页面上进行相应的修改，最后在日志中添加一条记录。一个来源可能会涉及到 10 到 15 页的维基页面。我个人更喜欢一次只处理一个来源，并全程参与处理过程——我会阅读摘要，检查更新内容，并指导大语言模型应该强调哪些部分。不过，您也可以一次性处理多个来源，这样需要较少的监督工作。您可以根据自己的习惯来设计工作流程，并将该流程记录在 schema 中，以便后续使用。

**Query.** You ask questions against the wiki. The LLM searches for relevant pages, reads them, and synthesizes an answer with citations. Answers can take different forms depending on the question — a markdown page, a comparison table, a slide deck (Marp), a chart (matplotlib), a canvas. The important insight: **good answers can be filed back into the wiki as new pages.** A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history. This way your explorations compound in the knowledge base just like ingested sources do.  
提问吧。您可以向维基百科提出问题。大型语言模型会搜索相关的页面，阅读这些内容，然后生成带有引用信息的答案。根据问题的不同，答案的形式也可以多种多样——可以是 Markdown 格式的文字页面、比较表格、幻灯片演示、图表（使用 matplotlib 库绘制）、或者画布形式的内容。重要的是，优秀的答案可以被整理成新的页面保存在维基百科上。您提出的比较、分析，或是您发现的关联点——这些宝贵的内容不应该只是消失在聊天记录中。通过这种方式，您的探索成果就像被吸收到的知识一样，不断积累在知识库中。

**Lint.** Periodically, ask the LLM to health-check the wiki. Look for: contradictions between pages, stale claims that newer sources have superseded, orphan pages with no inbound links, important concepts mentioned but lacking their own page, missing cross-references, data gaps that could be filled with a web search. The LLM is good at suggesting new questions to investigate and new sources to look for. This keeps the wiki healthy as it grows.  
定期要求大语言模型对维基页面进行健康检查。寻找以下问题：不同页面之间的矛盾之处、被新资料取代但仍存在的过时内容、没有内部链接的孤立页面、虽然提到了重要概念但缺乏相应的页面说明、缺失的相互引用关系、以及可以通过网络搜索填补的数据空白。大语言模型擅长提出新的调查问题以及新的查找来源的方法。这样就能让维基页面在不断发展中保持健康状态。

## Indexing and logging 索引和日志记录

Two special files help the LLM (and you) navigate the wiki as it grows. They serve different purposes:  
两个特殊的文件有助于大语言模型（以及您）在维基百科平台不断扩展的过程中进行导航。这两个文件有着不同的功能：

**index.md** is content-oriented. It's a catalog of everything in the wiki — each page listed with a link, a one-line summary, and optionally metadata like date or source count. Organized by category (entities, concepts, sources, etc.). The LLM updates it on every ingest. When answering a query, the LLM reads the index first to find relevant pages, then drills into them. This works surprisingly well at moderate scale (~100 sources, ~hundreds of pages) and avoids the need for embedding-based RAG infrastructure.  
index.md 是一个以内容为导向的目录。它包含了维基中所有的内容——每个页面都配有链接、一行的摘要，并且可选提供一些元数据，如日期或来源数量等。页面按照不同类别进行分类，例如实体、概念、来源等。LLM 在每次更新内容时都会更新这个目录。在回答查询时，LLM 首先读取目录以找到相关的页面，然后深入查询这些页面。这种方法在中等规模的应用中效果非常好（大约 100 个来源，~数百页），并且避免了需要基于嵌入式的 RAG 基础设施。

**log.md** is chronological. It's an append-only record of what happened and when — ingests, queries, lint passes. A useful tip: if each entry starts with a consistent prefix (e.g. `## [2026-04-02] ingest | Article Title`), the log becomes parseable with simple unix tools — `grep "^## \[" log.md | tail -5` gives you the last 5 entries. The log gives you a timeline of the wiki's evolution and helps the LLM understand what's been done recently.  
log.md 是按时间顺序记录的日志。它只支持追加新条目，记录的是发生的时间以及相关的操作——比如哪些条目被添加了、哪些条目通过了检查等。一个有用的提示是：如果每条条目的开头都有相同的前缀（例如 `## [2026-04-02] ingest | Article Title` ），那么使用简单的 Unix 工具就可以解析这些日志；例如， `grep "^## \[" log.md | tail -5` 可以显示最近 5 条记录。这个日志记录了维基页面的发展过程，有助于 LLM 理解最近发生了哪些事情。

## Optional: CLI tools 可选：CLI 工具

At some point you may want to build small tools that help the LLM operate on the wiki more efficiently. A search engine over the wiki pages is the most obvious one — at small scale the index file is enough, but as the wiki grows you want proper search. [qmd](https://github.com/tobi/qmd) is a good option: it's a local search engine for markdown files with hybrid BM25/vector search and LLM re-ranking, all on-device. It has both a CLI (so the LLM can shell out to it) and an MCP server (so the LLM can use it as a native tool). You could also build something simpler yourself — the LLM can help you vibe-code a naive search script as the need arises.  
在某个阶段，你可能需要构建一些小型工具，以帮助大型语言模型更高效地处理维基页面。在小型系统中，使用索引文件就足够了；但随着维基内容的不断增加，就需要具备完善的搜索功能了。qmd 就是一个不错的选择：它是一个本地化的 Markdown 文件搜索工具，采用混合的 BM25 向量搜索算法以及大型语言模型的重新排序功能，所有操作都在设备端完成。它既支持命令行接口，以便让大型语言模型通过它进行交互，也支持 MCP 服务器接口，使得大型语言模型可以将其作为原生工具来使用。当然，你也可以自己开发一个更简单的工具——在需要的时候，大型语言模型可以协助你编写一些简单的搜索脚本。

## Tips and tricks 一些小贴士和技巧

- **Obsidian Web Clipper** is a browser extension that converts web articles to markdown. Very useful for quickly getting sources into your raw collection.  
	Obsidian Web Clipper 是一款浏览器扩展程序，能够将网页文章转换为 Markdown 格式。这对于快速将网页内容添加到自己的素材收藏中非常有用。
- **Download images locally.** In Obsidian Settings → Files and links, set "Attachment folder path" to a fixed directory (e.g. `raw/assets/`). Then in Settings → Hotkeys, search for "Download" to find "Download attachments for current file" and bind it to a hotkey (e.g. Ctrl+Shift+D). After clipping an article, hit the hotkey and all images get downloaded to local disk. This is optional but useful — it lets the LLM view and reference images directly instead of relying on URLs that may break. Note that LLMs can't natively read markdown with inline images in one pass — the workaround is to have the LLM read the text first, then view some or all of the referenced images separately to gain additional context. It's a bit clunky but works well enough.  
	在本地下载图片。在 Obsidian 设置中，进入“文件和链接”选项，将“附件文件夹路径”设置为固定的目录（例如 `raw/assets/` ）。然后在设置中，查找“下载”功能，将其与快捷键绑定（例如 Ctrl+Shift+D）。当剪辑完一篇文章后，按下该快捷键即可将所有图片下载到本地磁盘上。这一功能虽然可选，但非常实用——它让 LLM 可以直接查看和引用图片，而无需依赖可能失效的 URL 链接。需要注意的是，LLM 无法直接一次性读取包含图片的 Markdown 文本；因此，解决方案是让 LLM 先阅读文本，然后单独查看部分或所有引用图片，以便获得更多上下文信息。虽然方法有些繁琐，但效果还是不错的。
- **Obsidian's graph view** is the best way to see the shape of your wiki — what's connected to what, which pages are hubs, which are orphans.  
	Obsidian 的图表视图是了解 wiki 结构的最佳方式——可以清楚地看到哪些内容是相互连接的，哪些页面是核心节点，哪些页面则属于孤立状态。
- **Marp** is a markdown-based slide deck format. Obsidian has a plugin for it. Useful for generating presentations directly from wiki content.  
	Marp 是一种基于 Markdown 的幻灯片格式。Obsidian 平台提供了相应的插件来支持这种格式。它非常适用于直接从 wiki 内容生成演示文稿。
- **Dataview** is an Obsidian plugin that runs queries over page frontmatter. If your LLM adds YAML frontmatter to wiki pages (tags, dates, source counts), Dataview can generate dynamic tables and lists.  
	Dataview 是 Obsidian 插件，它能够对页面前言部分进行查询处理。如果你的大语言模型在维基页面上添加了 YAML 格式的前言信息（如标签、日期、引用次数等），那么 Dataview 就可以生成动态的表格和列表。
- The wiki is just a git repo of markdown files. You get version history, branching, and collaboration for free.  
	这个维基页面只是一个包含 Markdown 文件的仓库而已。你可以免费获取版本历史、分支管理以及协作功能。

## Why this works 为什么这种方法有效？

The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.  
维护知识库的繁琐工作并不在于阅读或思考——而在于各种繁琐的更新工作。更新交叉引用、保持摘要的更新性、记录新数据何时与旧信息相矛盾、在数十页的内容中保持一致性……人们常常因为维护成本超过其价值而放弃维护维基百科。而大型语言模型则不会感到无聊，也不会忘记更新交叉引用；它们一次就能处理 15 个文件。维基百科能够持续被维护下去，因为其维护成本几乎为零。

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.  
人类的职责是筛选信息来源、指导分析过程、提出有价值的问题，并思考这一切背后的意义。而大型语言模型则负责处理所有其他的工作。

The idea is related in spirit to Vannevar Bush's Memex (1945) — a personal, curated knowledge store with associative trails between documents. Bush's vision was closer to this than to what the web became: private, actively curated, with the connections between documents as valuable as the documents themselves. The part he couldn't solve was who does the maintenance. The LLM handles that.  
这个想法在理念上与 Vannevar Bush 在 1945 年提出的 Memex 概念颇为相似——这是一种个人化的、经过精心管理的知识库，其中文档之间存在着关联联系。Bush 的设想更接近于这种模式，而不是现在的网络形式：网络应该是私密且由用户主动管理的，文档之间的连接与文档本身一样具有价值。不过，Bush 未能解决的是如何维护这个系统的问题。而 LLM 可以负责解决这个问题。

## Note 注意：

This document is intentionally abstract. It describes the idea, not a specific implementation. The exact directory structure, the schema conventions, the page formats, the tooling — all of that will depend on your domain, your preferences, and your LLM of choice. Everything mentioned above is optional and modular — pick what's useful, ignore what isn't. For example: your sources might be text-only, so you don't need image handling at all. Your wiki might be small enough that the index file is all you need, no search engine required. You might not care about slide decks and just want markdown pages. You might want a completely different set of output formats. The right way to use this is to share it with your LLM agent and work together to instantiate a version that fits your needs. The document's only job is to communicate the pattern. Your LLM can figure out the rest.  
此文档旨在保持抽象性。它描述了概念框架，而非具体的实现方式。实际的目录结构、模式规范、页面格式以及所使用的工具等细节，都将取决于你的领域需求、个人偏好以及你所选择的大型语言模型。上述所有内容都是可选的，你可以选择使用那些对你有用的功能，而忽略那些不需要的功能。例如，如果你只使用文本形式来展示内容，那么就完全不需要处理图片；如果你的知识库规模较小，那么可能只需要索引文件即可，无需搜索引擎；如果你不擅长使用幻灯片形式展示内容，那么就可以使用 Markdown 格式来编写页面；或者，你可能会希望采用完全不同的输出格式。正确的使用方式是将此文档与你的大型语言模型相结合，共同打造出符合你需求的版本。文档的唯一作用就是传达出这种设计模式，剩下的事情就由你的语言模型来处理了。