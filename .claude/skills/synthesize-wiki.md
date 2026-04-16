---
name: synthesize-wiki
description: Use this skill when the user says "synthesize", "write a report on X", "synthesis", "pull together everything in the wiki about X", or "create a synthesis report on X". Proactively synthesizes a specified topic across multiple wiki pages, generates a structured report, and writes it directly to wiki/syntheses/ — no second confirmation needed.
---

# synthesize-wiki

Proactively synthesize Wiki knowledge across multiple pages, generate a structured report, and write it directly to `wiki/syntheses/`.

> Difference from query-wiki: query is "answer first, then passively archive"; synthesize is "define the output target first, then write directly".

## Steps

1. **Confirm synthesis topic**
   - If the instruction is vague (e.g. "synthesize things"), ask: "Please confirm the synthesis topic and desired depth (overview / deep analysis / comparison report)?"
   - If the topic is clear, proceed to step 2

2. **Scan relevant Wiki pages**
   - Read `wiki/index.md` for global view
   - Read `wiki/hot.md` for current focus
   - Read by relevance: `wiki/sources/`, `wiki/concepts/`, `wiki/entities/`, `wiki/comparisons/`, `wiki/syntheses/` (avoid duplicating existing syntheses)

3. **Identify source coverage**
   - Flag points supported by **≥2 sources** (higher confidence)
   - Flag points from **only 1 source** (note accordingly)
   - Flag **gaps or unresolved open questions** in the Wiki

4. **Generate synthesis report**

   Report structure (adjust flexibly based on topic):
   ```
   ## Core Conclusions (2–3 sentences)

   ## Key Findings
   ### {Sub-topic 1}
   ### {Sub-topic 2}
   ...

   ## Cross-Source Consensus
   - {point} (sources: [[page A]], [[page B]])

   ## Single-Source Points (pending verification)
   - {point} (source: [[page C]])

   ## Open Questions / Wiki Gaps
   - {description} → suggested sources to add

   ## Referenced Pages
   - [[wiki/sources/...]]
   - [[wiki/concepts/...]]
   ```

5. **Write to `wiki/syntheses/{slug}-{YYYY-MM-DD}.md`**

   Frontmatter template:
   ```yaml
   ---
   title: "{synthesis topic}"
   created: {today's date}
   modified: {today's date}
   tags: [#synthesis, #{topic tag}]
   related: [{all referenced wiki pages}]
   summary: "{one-sentence summary}"
   sources_count: {number of referenced sources}
   ---
   ```

6. **Update `wiki/index.md`**
   - Add entry to Syntheses section, update count

7. **Add reverse links in referenced pages**
   - Append to the bottom of each referenced wiki page: `- [[wiki/syntheses/{slug}-{date}.md]] (synthesis)`

8. **Append to `wiki/log.md`**
   ```
   ## [{date}] synthesize | {topic}
   - Saved as: wiki/syntheses/{slug}-{date}.md
   - Referenced sources: {page list}
   - Identified gaps: {open question list, or "none"}
   ```

9. **Update `VAULT-INDEX.md`**
   - Increment Syntheses count by 1
   - Append to recent activity

10. **Update `wiki/hot.md`**
    - Append to recent activity:
      ```
      - {date} synthesize | {topic} (referenced {n} pages, identified {m} gaps)
      ```

11. **Git sync**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "synthesize: {topic}"
    git push
    ```

## Output

Display the full synthesis report in the conversation, and report: write path, number of referenced sources, number of identified open questions, Git push status.
