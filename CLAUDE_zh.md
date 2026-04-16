# CLAUDE.md — Wiki 知识库 Schema

## 概述
这是一个由 LLM 维护的个人知识库 Wiki。Wiki 随每次 ingest 和 query 积累知识，越来越丰富。

**核心原则**：人类负责策划来源和提问，LLM 负责所有 bookkeeping 工作：总结、交叉引用、归档和维护一致性。

## 目录结构

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

## 四种核心操作

> **强制规则：收到操作指令后，必须先读取对应 skill 文件的完整内容，再开始执行。不得跳过此步骤。**

### Ingest（摄入）
**触发词**："Ingest raw/..."、"摄入"、"把这个加入知识库"、"存到知识库"、"记录到知识库"、"加到知识库"、"存进知识库"、"放入知识库"、"更新知识库"、"帮我存一下"、"处理这篇文章"，或用户直接发来文件/文本内容并提到知识库
**Skill 文件**：`.claude/skills/ingest-wiki.md`
**说明**：支持两种方式触发：用户指定 raw/ 路径，或直接发来内容由 LLM 自动归档到 raw/。提取要点，整合进 Wiki，更新 index / concepts / entities / comparisons / log / VAULT-INDEX，Git 提交。

### Query（查询）
**触发词**："根据 wiki"、"查询 wiki"、"知识库里有没有"、"wiki 里怎么说"，以及涉及已收录主题的实质性提问
**Skill 文件**：`.claude/skills/query-wiki.md`
**说明**：从 index.md 定位相关页，综合多页内容给出带来源标注的答案，达到归档标准时询问是否保存回 wiki/syntheses/。

### Synthesize（综合）
**触发词**："综合一下"、"帮我写一篇关于 X 的报告"、"Synthesize"、"synthesis"、"把 wiki 里关于 X 的内容整合一下"、"生成综合报告"
**Skill 文件**：`.claude/skills/synthesize-wiki.md`
**说明**：主动跨多个 wiki 页面综合指定主题，生成结构化报告直接写入 wiki/syntheses/，更新 index / log / VAULT-INDEX，Git 提交。

### Lint（健康检查）
**触发词**："Lint the wiki"、"lint wiki"、"健康检查"、"检查知识库"、"wiki 有没有问题"
**Skill 文件**：`.claude/skills/lint-wiki.md`
**说明**：扫描矛盾、孤立页、缺失概念、过时声明、结构问题，生成诊断报告，追加 log，Git 提交。

## 约定

1. **严格的层级边界**
   - `raw/` 存放原始来源：人类手动写入，或由 LLM 在 Ingest 时代为归档，LLM 不主动修改已有内容
   - `wiki/` 只由 LLM 写入和维护
2. **每次操作都追加到 wiki/log.md**
3. **创建新页面后必须更新 wiki/index.md**
4. **使用双链语法 [[page]] 进行内部引用**
5. **VAULT-INDEX.md 由 LLM 自动维护**，每次 ingest/lint/synthesize 后更新统计和活动

## 页面类型创建规则

| 页面类型 | 位置 | 创建时机 |
|----------|------|----------|
| sources/ | `wiki/sources/` | 每次 ingest 必创建，一篇来源对应一个摘要页 |
| concepts/ | `wiki/concepts/` | ingest 时识别到新概念；或 synthesize 时产出概念级结论 |
| entities/ | `wiki/entities/` | ingest 时识别到新人物 / 公司 / 产品 / 组织 |
| comparisons/ | `wiki/comparisons/` | ingest 时新来源与已有内容存在 **≥3 个对比维度**的竞争/替代关系（如两种工具、两种方法论） |
| syntheses/ | `wiki/syntheses/` | 用户主动触发 synthesize 操作；或 query 答案满足归档标准（综合 >2 个来源、>200 字）且用户确认归档 |

## Git 工作流

每次 Ingest / Lint 操作完成后：

1. `git add .`
2. `git commit -m "{操作类型}: {简短描述}"`
3. `git push`
