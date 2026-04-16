---
name: query-wiki
description: 当用户提问、查询、或需要基于知识库内容回答问题时使用此 skill。触发词包括："根据 wiki"、"查询 wiki"、"知识库里有没有"、"wiki 里怎么说"，以及任何涉及知识库已收录主题的实质性问题。负责从 index.md 定位相关页面、综合多页内容给出答案、并在答案有价值时归档回 wiki。
---

# query-wiki

基于 Wiki 知识库内容回答问题，综合多个页面信息，可选归档有价值的答案。

## 执行步骤

1. **读取 `wiki/index.md`**
   - 获取全局概览和所有页面列表
   - 确定与问题相关的页面

2. **优先读取 `wiki/hot.md`**
   - 了解当前会话焦点，判断问题是否在热点范围内

3. **阅读相关页面**
   - 按相关度从高到低读取 `wiki/sources/`、`wiki/concepts/`、`wiki/entities/`、`wiki/comparisons/`、`wiki/syntheses/` 中的相关页面
   - 如信息不足，读取更多关联页面

4. **综合答案**
   - 基于 Wiki 内容回答，不引入 Wiki 外的未经收录信息
   - 用 `[[wiki/sources/xxx]]` 格式标注每个信息点的来源页面
   - 如 Wiki 中信息不足，明确说明："Wiki 中暂无此信息，以下基于通用知识作答"

5. **判断是否归档**

   归档标准（满足任一条即询问用户是否归档）：
   - 答案是综合多个来源的分析（>200 字）
   - 答案包含对比表格或结构化框架
   - 答案揭示了 Wiki 中未显式记录的关联或洞见

   不归档的情况：
   - 简单事实查询（一句话能答完）
   - 答案与现有某个 Wiki 页面高度重复

   归档位置：
   - 综合分析 → `wiki/syntheses/`
   - 新概念阐释 → `wiki/concepts/`
   - 文件命名：`{slug}-{YYYY-MM-DD}.md`

   归档 frontmatter 模板：
   ```yaml
   ---
   title: "{问题或主题}"
   created: {今日日期}
   modified: {今日日期}
   tags: [#query, #{主题标签}]
   related: [{引用的来源页面}]
   summary: "{一句话摘要}"
   query: "{原始问题}"
   ---
   ```

6. **归档后更新（仅当确认归档时执行）**
   - 更新 `wiki/index.md`：在对应节新增条目
   - 在已引用的 wiki 页面中反向添加 `[[新页面]]` 链接
   - 追加到 `wiki/log.md`：
     ```
     ## [{日期}] query | {问题摘要}
     - 查询: {原始问题}
     - 保存为: wiki/syntheses/{slug}.md
     - 引用来源: {列表}
     ```
   - 更新 `wiki/hot.md`：在"最近活动"区块追加：
     ```
     - {日期} query | 归档：{页面标题}（来源：{引用页面列表}）
     ```
   - **Git 同步**：
     ```bash
     # 如果知识库不在 ~/knowledge-vault，请将下面路径替换为实际路径
     cd ~/knowledge-vault
     git add .
     git commit -m "query: {问题摘要}"
     git push
     ```

## 完成后输出

直接回答问题，所有信息点标注来源，如达到归档标准则询问用户是否保存。
