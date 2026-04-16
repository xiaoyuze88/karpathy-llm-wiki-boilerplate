---
title: "LLM Wiki 范式"
created: 2026-04-16
modified: 2026-04-16
tags: [#concept, #llm-wiki, #knowledge-management, #pkm]
related:
  - "[[wiki/concepts/rag-vs-llm-wiki.md]]"
  - "[[wiki/concepts/knowledge-compaction.md]]"
  - "[[wiki/entities/andrej-karpathy.md]]"
  - "[[wiki/sources/summary-karpathy-llm-wiki.md]]"
summary: "以 LLM 增量构建并维护持久 Wiki 的知识管理范式：知识编译一次、永久复用，LLM 负责全部簿记工作。"
---

## 定义

LLM Wiki 范式由 Andrej Karpathy 于 2026 年 4 月提出，核心命题：**停止在每次查询时重新推导知识，将其一次性编译成结构化 Wiki，让 LLM 处理导致人类放弃知识库的繁琐簿记工作。**

## 核心原则

1. **知识编译一次**：新来源添加时，LLM 提取关键信息整合进 Wiki，不再每次查询重新推导
2. **复合增长**：每添加一个来源、每问一个问题，Wiki 都变得更丰富
3. **人机分工**：人类策划来源、提出问题；LLM 负责摘要、交叉引用、一致性维护
4. **持久化**：知识以结构化 Markdown 持久存储，跨会话可用

## 三层架构

| 层级 | 内容 | 职责 |
|------|------|------|
| **第一层**（raw/） | 原始来源：文章、论文、转录 | 不可变，LLM 只读 |
| **第二层**（wiki/） | LLM 生成的 Markdown 文件 | LLM 完全负责维护 |
| **第三层**（Schema） | CLAUDE.md / AGENTS.md | 定义结构约定和工作流 |

## 三大核心操作

- **Ingest（摄入）**：读取原始来源 → 提取要点 → 创建/更新 Wiki 页面
- **Query（查询）**：针对 Wiki 提问，LLM 综合相关页面回答，好答案可归档为新页面
- **Lint（健康检查）**：扫描孤立页面、矛盾、过时内容，出报告后等用户确认再修复

## 经典比喻

> Obsidian 是 IDE；LLM 是程序员；Wiki 是代码库。

思想渊源：Vannevar Bush 的 **Memex（1945）**——个人精心策划的知识存储，文档间有关联轨迹。LLM Wiki 解决了 Memex 遗留的"谁来维护"问题。

## 规模边界

- 可靠运行上限：约 **100-200 篇**来源、**50,000-100,000 Token**
- 超出后：index.md 无法装入上下文窗口，需转向 RAG 或混合架构

## 来源

- [[wiki/sources/summary-karpathy-llm-wiki.md]]（2026-04-16）
