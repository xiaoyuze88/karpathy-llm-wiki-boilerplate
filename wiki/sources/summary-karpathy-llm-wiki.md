---
title: "Karpathy LLM Wiki"
created: 2026-04-16
modified: 2026-04-16
tags: [#source, #llm-wiki, #knowledge-management, #rag, #pkm]
related:
  - "[[wiki/concepts/llm-wiki-paradigm.md]]"
  - "[[wiki/concepts/rag-vs-llm-wiki.md]]"
  - "[[wiki/concepts/knowledge-compaction.md]]"
  - "[[wiki/entities/andrej-karpathy.md]]"
summary: "Karpathy 提出以 LLM 增量构建并维护持久 Wiki 的范式，用编译时知识积累替代 RAG 的查询时重新推导。"
source_path: "raw/articles/Karpathy_LLM_Wiki.md"
---

## 摘要

2026 年 4 月，Andrej Karpathy 发布 LLM Wiki Gist，提出一种个人知识管理新范式：**知识编译一次，永久复用**，由 LLM 负责所有簿记工作，人类只需策划来源和提出问题。该方案迅速获得 5,000+ Stars，并催生数十个开源实现。

## 关键要点

1. **编译时 vs 查询时**：LLM Wiki 在摄入阶段将知识编译进结构化 Markdown，之后查询直接复用；RAG 每次查询都从原始文档重新检索合成，无积累效应。

2. **三层架构**：
   - 第一层：`raw/`（原始来源，LLM 只读）
   - 第二层：`wiki/`（LLM 编写的结构化知识库）
   - 第三层：Schema（CLAUDE.md/AGENTS.md，定义工作流和约定）

3. **规模临界点**：可靠运行上限约 100-200 篇来源、50,000-100,000 Token。超出此范围 index.md 无法装入上下文窗口，必须转向 RAG 或混合架构。

4. **三大操作**：Ingest（摄入）、Query（查询）、Lint（健康检查）。其中 Lint 生成诊断报告但不自动修改，需用户确认。

5. **规模化三大缺陷**：
   - 表达层：wikilink 缺乏语义（解决方案：obsidian-wikilink-types `@syntax`）
   - 发现层：关系发现依赖手动（解决方案：AI 代理 Vault Linker）
   - 持久层：知识困在单机（解决方案：Penfield MCP 持久化知识图谱）

## 重要引用

> "知识编译一次，然后保持最新，而不是每次查询重新推导。"

> "Obsidian 是 IDE；LLM 是程序员；Wiki 是代码库。"

> "规模决定架构，治理决定结果。"

## 与 Wiki 的关联

- 本文是本知识库的**元来源**——描述的正是本知识库的运作方式
- 提出的三层架构与本项目目录结构直接对应
- 规模临界点（100-200 篇）是本知识库扩展的重要参考边界
