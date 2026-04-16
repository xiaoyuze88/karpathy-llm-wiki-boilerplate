---
name: query-wiki
description: Use this skill when the user asks a question, queries the wiki, or needs an answer grounded in the knowledge base. Trigger phrases include "according to the wiki", "look up X in the wiki", "what does the wiki say about", "is X in the wiki", as well as any substantive question about topics already in the knowledge base. Responsible for locating relevant pages from index.md, synthesizing multi-page content into a sourced answer, and optionally archiving high-value answers to wiki/syntheses/ or wiki/concepts/.
---

# query-wiki

Answer questions based on Wiki knowledge base content, synthesizing information from multiple pages, with optional archival of valuable answers.

## Steps

1. **Read `wiki/index.md`**
   - Get the global overview and full page list
   - Identify pages relevant to the question

2. **Read `wiki/hot.md` first**
   - Understand the current session focus, determine if the question falls within hot topics

3. **Read relevant pages**
   - Read pages from `wiki/sources/`, `wiki/concepts/`, `wiki/entities/`, `wiki/comparisons/`, `wiki/syntheses/` in order of relevance
   - If information is insufficient, read additional linked pages

4. **Synthesize the answer**
   - Answer based on Wiki content; do not introduce information not yet in the Wiki
   - Annotate each piece of information with its source page using `[[wiki/sources/xxx]]` format
   - If Wiki information is insufficient, clearly state: "The Wiki does not contain this information yet; the following is based on general knowledge"

5. **Decide whether to archive**

   Archive criteria (ask the user if any one is met):
   - Answer synthesizes **>2 sources** with analysis (>200 words)
   - Answer includes a comparison table or structured framework
   - Answer reveals connections or insights not explicitly recorded in the Wiki

   Do not archive if:
   - Simple factual query (answerable in one sentence)
   - Answer is largely duplicative of an existing Wiki page

   **Archive location** (choose one based on content):
   - Cross-source synthesis, analytical answer → `wiki/syntheses/{slug}-{YYYY-MM-DD}.md`
   - Answer clarifies a new concept or meaningfully extends an existing one → `wiki/concepts/{slug}.md`

   Archive frontmatter template:
   ```yaml
   ---
   title: "{question or topic}"
   created: {today's date}
   modified: {today's date}
   tags: [#query, #{topic tag}]
   related: [{referenced source pages}]
   summary: "{one-sentence summary}"
   query: "{original question}"
   ---
   ```

6. **Post-archive updates (only when archival is confirmed)**
   - Update `wiki/index.md`: add entry to the relevant section (Syntheses or Concepts), update count
   - Add reverse `[[new page]]` links in all referenced wiki pages
   - Append to `wiki/log.md`:
     ```
     ## [{date}] query | {question summary}
     - Query: {original question}
     - Saved as: wiki/syntheses/{slug}.md or wiki/concepts/{slug}.md
     - Referenced sources: {list}
     ```
   - Update `wiki/hot.md`, append to "Recent Activity":
     ```
     - {date} query | archived: {page title} (sources: {list})
     ```
   - Update `VAULT-INDEX.md`: increment corresponding count by 1
   - **Git sync**:
     ```bash
     cd ~/knowledge-vault
     git add .
     git commit -m "query: {question summary}"
     git push
     ```

## Output

Answer the question directly with all information points sourced; if archival criteria are met, ask the user whether to save.
