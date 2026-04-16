---
name: ingest-wiki
description: 当用户说 "Ingest raw/..."、"摄入"、"处理这篇文章"、"把这个加入知识库"、"存到知识库"、"记录到知识库"、"加到知识库"、"存进知识库"、"放入知识库"、"更新知识库"、"帮我存一下"、"ingest" 等，或直接把文件/文本内容发给模型并提到知识库时，使用此 skill。包含：原始文件归档到 raw/、摘要提取、概念/实体/comparisons 页更新、index/log/VAULT-INDEX 维护、Git 提交全流程。
---

# ingest-wiki

将来源内容读取并整合进 Wiki 知识库。支持两种触发方式：
- **方式 A**：用户已将文件放入 `raw/`，直接指定路径（如 `Ingest raw/articles/foo.md`）
- **方式 B**：用户直接发来文件或文本内容，由模型先归档到 `raw/`，再执行后续流程

## 执行步骤

1. **确定源文件路径**

   **方式 A（用户指定路径）**：直接使用用户提供的 `raw/` 路径，跳到步骤 2。

   **方式 B（用户直接发来内容）**：
   - 根据内容类型自动判断归档子目录：
     - 文章/博客/网页 → `raw/articles/`
     - 学术论文 → `raw/papers/`
     - 仓库 README / 技术文档 → `raw/repos/`
     - 会议/播客转录 → `raw/transcripts/`
     - 数据文件 → `raw/data/`
   - 文件名直接复用用户发来的原始文件名（如 `foo.md`）
   - 若目标路径已存在同名文件，**暂停并询问用户**：
     ```
     raw/{子目录}/{文件名} 已存在，请选择：
     1. 覆盖原有文件
     2. 新增为 {文件名-2}.md（数字后缀递增）
     ```
   - 按用户选择写入文件后继续
   - 先读 `wiki/index.md` 获取现有 Wiki 全局视图，再读 `wiki/hot.md` 了解当前焦点

2. **分析内容，提取 3-5 个关键要点**
   - 核心论点是什么？
   - 涉及哪些概念（需要创建/更新 `wiki/concepts/` 页面）？
   - 涉及哪些实体（人物/公司/产品，需要创建/更新 `wiki/entities/` 页面）？
   - 与已有 Wiki 内容有哪些关联或矛盾？
   - 与 `wiki/hot.md` 中的当前焦点是否相关？
   - **comparisons 判断**：新来源是否与已有某个概念/工具存在 **≥3 个可对比维度**的竞争或替代关系（如两种框架、两种方法论）？若是，在步骤 5a 创建 `wiki/comparisons/` 页面而非将对比内容塞入 concepts。

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
   - 如有新比较页，在 Comparisons 节增加条目，更新计数

5. **更新相关概念页（`wiki/concepts/*.md`）**
   - 不存在则创建（含完整 frontmatter）
   - 已存在则在末尾追加新来源的视角，更新 `modified` 日期

5a. **创建比较页（如步骤 2 判断需要，`wiki/comparisons/*.md`）**
   - 文件命名：`{slug-a}-vs-{slug-b}.md`
   - 内容结构：对比维度表格 + 各维度详述 + 适用场景 + 结论建议
   - frontmatter tags 加 `#comparison`
   - 同时在对应的 concepts 页末尾添加指向此比较页的链接

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
   - 新增比较页: {列表，无则写"无"}
   - 矛盾: {描述，无则写"无"}
   ```

9. **更新 `VAULT-INDEX.md`**
   - Sources 统计 +1
   - 最近活动列表追加本次 ingest 记录

10. **更新 `wiki/hot.md`**
    - 在"最近活动"区块追加本次摘要：
      ```
      - {日期} ingest | {来源文件名}：{1-2句关键要点}，新增页面：{列表}
      ```
    - 如 hot.md 超过 500 字，压缩最早的活动记录（保留结构，删减细节）

11. **Git 同步**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "ingest: {文件名}"
    git push
    ```

## 完成后输出

汇报：创建/更新了哪些文件、发现的 3-5 个关键要点、是否有矛盾、Git push 是否成功。
