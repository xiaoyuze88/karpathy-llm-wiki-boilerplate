---
title: "知识压缩（Knowledge Compaction）"
created: 2026-04-16
modified: 2026-04-16
tags: [#concept, #llm-wiki, #maintenance, #scalability]
related:
  - "[[wiki/concepts/llm-wiki-paradigm.md]]"
  - "[[wiki/sources/summary-karpathy-llm-wiki.md]]"
summary: "仅追加的 Wiki 最终会变成它原本要替代的混乱——压缩（Compaction）是解决增长悖论的核心机制。"
---

## 问题：增长悖论

**仅追加的 Wiki 存在根本性矛盾**：

> 随着来源不断增加，Wiki 内容只增不减，最终会变成它原本要替代的混乱——一个庞大、难以导航的信息堆。

这是 LLM Wiki 社区的核心批判之一：如果永远只"追加"，知识库的维护难度会随规模线性增长，重蹈传统 Wiki 被废弃的覆辙。

## 压缩的核心思路

压缩（Compaction）指**定期对 Wiki 进行结构性精简**：

- 合并重复或高度相似的概念页
- 将多个小条目整合进更高层级的综合页
- 删除已过时或被更新来源覆盖的内容
- 重构关联稀疏的孤立页面

## 与 Lint 操作的关系

Lint（健康检查）是压缩的**诊断阶段**：
- Lint 扫描矛盾、孤立页面、过时内容，生成报告
- 报告需用户确认后才执行修复
- 压缩是在 Lint 诊断基础上的更深度结构重组

## 实践建议

- 按**目录**而非整体运行 Lint，避免噪音
- 知识主题目录（概念、综合）是压缩真正发挥价值的地方
- 工具/备忘录目录每篇独立，报"孤立页面"意义不大
- 当 wiki/index.md 出现导航困难时，即触发压缩时机

## 来源

- [[wiki/sources/summary-karpathy-llm-wiki.md]]（2026-04-16）
