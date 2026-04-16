---
name: ingest-wiki
description: 当用户说 "Ingest raw/..."、"摄入 raw/..."、"处理这篇文章"、"把这个加入知识库"、"ingest" 等，需要将 raw/ 目录下的新来源文件读取并整合进 Wiki 知识库时，使用此 skill。覆盖：摘要提取、概念/实体页更新、index/log/VAULT-INDEX 维护、Git 提交全流程。
---

# ingest-wiki

将 `raw/` 下的来源文件读取并整合进 Wiki 知识库。

## 执行步骤

1. **读取源文件**
   - 路径：用户指定的 `raw/` 下文件（支持 .md、.txt）
   - 先读 `wiki/index.md` 获取现有 Wiki 全局视图，再读 `wiki/hot.md` 了解当前焦点

2. **分析内容，提取 3-5 个关键要点**
   - 核心论点是什么？
   - 涉及哪些概念（需要创建/更新 `wiki/concepts/` 页面）？
   - 涉及哪些实体（人物/公司/产品，需要创建/更新 `wiki/entities/` 页面）？
   - 与已有 Wiki 内容有哪些关联或矛盾？
   - 与 `wiki/hot.md` 中的当前焦点是否相关？

3. **创建 `wiki/sources/summary-{slug}.md`**
   ```yaml
   ---
   title: "{原文标题}"
   created: {今日日期}
   modified: {今日日期}
   tags: [#source, #{主题标签}]
   related: []
   summary: "{一句话摘要}"
   source_path: "raw/..."
   ---
   ```
   内容包含：摘要、关键要点列表、重要引用、与已有 Wiki 的关联/矛盾说明

4. **更新 `wiki/index.md`**
   - Sources 节增加条目，更新计数
   - 如有新概念，在 Concepts 节增加条目，更新计数
   - 如有新实体，在 Entities 节增加条目，更新计数

5. **更新相关概念页（`wiki/concepts/*.md`）**
   - 不存在则创建（含完整 frontmatter）
   - 已存在则在末尾追加新来源的视角，更新 `modified` 日期

6. **更新相关实体页（`wiki/entities/*.md`）**
   - 同步骤 5

7. **标记矛盾（如有）**
   - 在摘要页和相关概念/实体页中用 `> ⚠️ 矛盾：...` 标注

8. **追加到 `wiki/log.md`**
   ```
   ## [{日期}] ingest | {文件名}
   - 创建: wiki/sources/summary-{slug}.md
   - 更新: wiki/index.md
   - 新增概念: {列表，无则写"无"}
   - 更新概念: {列表，无则写"无"}
   - 新增实体: {列表，无则写"无"}
   - 更新实体: {列表，无则写"无"}
   - 矛盾: {描述，无则写"无"}
   ```

9. **更新 `VAULT-INDEX.md`**
   - Sources 统计 +1
   - 最近活动列表追加本次 ingest 记录

10. **Git 同步**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "ingest: {文件名}"
    git push
    ```

## 完成后输出

汇报：创建/更新了哪些文件、发现的 3-5 个关键要点、是否有矛盾、Git push 是否成功。
