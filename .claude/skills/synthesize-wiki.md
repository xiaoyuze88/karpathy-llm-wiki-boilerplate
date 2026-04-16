---
name: synthesize-wiki
description: 当用户说"综合一下"、"帮我写一篇关于 X 的报告"、"Synthesize"、"synthesis"、"把 wiki 里关于 X 的内容整合一下"、"生成综合报告"时使用此 skill。主动跨多个 wiki 页面综合指定主题，生成结构化报告并直接写入 wiki/syntheses/，无需等用户二次确认归档。
---

# synthesize-wiki

主动跨多页综合 Wiki 知识，生成结构化报告，直接写入 `wiki/syntheses/`。

> 与 query-wiki 的区别：query 是"先回答、再被动归档"；synthesize 是"先明确产出目标、直接写入"。

## 执行步骤

1. **确认综合主题**
   - 如用户指令模糊（如"综合一下"），询问："请确认综合主题和期望深度（概述 / 深度分析 / 对比报告）？"
   - 主题明确则直接进入步骤 2

2. **扫描相关 Wiki 页面**
   - 读取 `wiki/index.md` 获取全局视图
   - 读取 `wiki/hot.md` 了解当前焦点
   - 按主题相关性读取：`wiki/sources/`、`wiki/concepts/`、`wiki/entities/`、`wiki/comparisons/`、`wiki/syntheses/`（已有综合避免重复）

3. **识别来源覆盖情况**
   - 标注哪些观点来自 **≥2 个来源**（较高可信度）
   - 标注哪些观点仅来自 **1 个来源**（需注明）
   - 标注 Wiki 中存在的**空白或未解决的开放问题**

4. **生成综合报告**

   报告结构（根据主题灵活调整）：
   ```
   ## 核心结论（2-3 句）

   ## 主要发现
   ### {子主题 1}
   ### {子主题 2}
   ...

   ## 跨来源共识
   - {观点}（来源：[[页面A]]、[[页面B]]）

   ## 单来源观点（待补充验证）
   - {观点}（来源：[[页面C]]）

   ## 开放问题 / Wiki 空白
   - {问题描述} → 建议补充来源方向

   ## 引用页面
   - [[wiki/sources/...]]
   - [[wiki/concepts/...]]
   ```

5. **写入 `wiki/syntheses/{slug}-{YYYY-MM-DD}.md`**

   frontmatter 模板：
   ```yaml
   ---
   title: "{综合主题}"
   created: {今日日期}
   modified: {今日日期}
   tags: [#synthesis, #{主题标签}]
   related: [{所有引用的 wiki 页面}]
   summary: "{一句话摘要}"
   sources_count: {引用来源数量}
   ---
   ```

6. **更新 `wiki/index.md`**
   - Syntheses 节增加条目，更新计数

7. **在引用页面中添加反向链接**
   - 在每个被引用的 wiki 页面末尾追加：`- [[wiki/syntheses/{slug}-{date}.md]]（综合报告）`

8. **追加到 `wiki/log.md`**
   ```
   ## [{日期}] synthesize | {主题}
   - 保存为: wiki/syntheses/{slug}-{date}.md
   - 引用来源: {页面列表}
   - 识别空白: {开放问题列表，无则写"无"}
   ```

9. **更新 `VAULT-INDEX.md`**
   - Syntheses 统计 +1
   - 最近活动追加本次记录

10. **更新 `wiki/hot.md`**
    - 最近活动追加：
      ```
      - {日期} synthesize | {主题}（引用 {n} 个页面，识别 {m} 个空白）
      ```

11. **Git 同步**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "synthesize: {主题}"
    git push
    ```

## 完成后输出

在对话中展示综合报告全文，并汇报：写入路径、引用来源数、识别的开放问题数、Git push 状态。
