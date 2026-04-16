# CLAUDE.md — Wiki Knowledge Base Schema

## Overview
这是一个由 LLM 维护的个人知识库 Wiki。Wiki 随每次 ingest 和 query 积累知识，越来越丰富。

**核心原则**：人类负责策划来源和提问，LLM 负责所有 bookkeeping 工作：总结、交叉引用、归档和维护一致性。

## Directory Structure

```
.
├── VAULT-INDEX.md       # 实时仪表板（自动维护）
├── CLAUDE.md            # 本文件
├── raw/                    # Layer 1: 原始数据（人类写，LLM 只读）
│   ├── articles/          # 网页剪藏文章
│   ├── papers/            # 学术论文、PDF
│   ├── repos/            # 仓库 README
│   ├── transcripts/       # 会议/播客转录
│   ├── data/             # CSV、JSON 数据文件
│   └── assets/           # 图片、附件
│
└── wiki/                  # Layer 2: LLM 维护的知识库（LLM 写和维护）
    ├── index.md         # 主目录（查询从这里开始）
    ├── log.md           # 活动时间线（只追加）
    ├── hot.md           # 热缓存（会话上下文）
    ├── sources/         # 源摘要页
    ├── entities/        # 人物/公司/产品页
    ├── concepts/        # 概念解释页
    ├── comparisons/      # 比较分析页
    └── syntheses/       # 综合论文/报告
```

## Wiki 页面格式

### 必需 Frontmatter
```yaml
---
title: "页面标题"
created: 2026-04-14
modified: 2026-04-14
tags: [#concept, #ai, #llm]
related: [[wiki/concepts/other-concept]], [[wiki/entities/person-x]]
summary: "一句话摘要"
---
```

## 三种核心操作

> **强制规则：收到操作指令后，必须先读取对应 skill 文件的完整内容，再开始执行。不得跳过此步骤。**

### Ingest（摄入）
**触发词**："Ingest raw/..."、"摄入"、"把这个加入知识库"、"处理这篇文章"
**Skill 文件**：`.claude/skills/ingest-wiki.md`
**说明**：读取 raw/ 来源文件，提取要点，整合进 Wiki，更新 index / concepts / entities / log / VAULT-INDEX，Git 提交。

### Query（查询）
**触发词**："根据 wiki"、"查询 wiki"、"知识库里有没有"、"wiki 里怎么说"，以及涉及已收录主题的实质性提问
**Skill 文件**：`.claude/skills/query-wiki.md`
**说明**：从 index.md 定位相关页，综合多页内容给出带来源标注的答案，达到归档标准时询问是否保存回 wiki。

### Lint（健康检查）
**触发词**："Lint the wiki"、"lint wiki"、"健康检查"、"检查知识库"、"wiki 有没有问题"
**Skill 文件**：`.claude/skills/lint-wiki.md`
**说明**：扫描矛盾、孤立页、缺失概念、过时声明、结构问题，生成诊断报告，追加 log，Git 提交。

## 约定

1. **严格的层级边界**
   - `raw/` 只由人类写入，LLM 只读
   - `wiki/` 只由 LLM 写入和维护
2. **每次操作都追加到 wiki/log.md**
3. **创建新页面后必须更新 wiki/index.md**
4. **使用双链语法 [[page]] 进行内部引用**
5. **VAULT-INDEX.md 由 LLM 自动维护**，每次 ingest/lint 后更新统计和活动

## Git 工作流

每次 Ingest / Lint 操作完成后：

1. `git add .`
2. `git commit -m "{操作类型}: {简短描述}"`
3. `git push`

**冲突处理**：
- push 失败时，先 `git pull --rebase` 尝试自动合并
- 如有冲突，**保留双方内容**（用 `<<<<<<<` / `>>>>>>>` 标记），不删除任何一方
- 标记冲突后告知用户，由用户决定保留哪个版本
