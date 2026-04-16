---
name: ingest-wiki
description: Use this skill when the user says "Ingest raw/...", "add to the wiki", "save to the knowledge base", "store in the wiki", "put this in the wiki", "process this article", "ingest", or when the user sends file/text content and mentions the wiki. Covers the full pipeline: archiving raw files, extracting summaries, updating concepts/entities/comparisons pages, maintaining index/log/VAULT-INDEX, and Git commit.
---

# ingest-wiki

Read source content and integrate it into the Wiki knowledge base. Supports two trigger modes:
- **Mode A**: User has already placed the file in `raw/` and provides the path directly (e.g. `Ingest raw/articles/foo.md`)
- **Mode B**: User sends file or text content directly; the LLM archives it to `raw/` first, then continues

## Steps

1. **Determine source file path**

   **Mode A (user-specified path)**: Use the provided `raw/` path directly and proceed to step 2.

   **Mode B (user sends content directly)**:
   - Auto-detect the appropriate subdirectory based on content type:
     - Articles / blog posts / web pages → `raw/articles/`
     - Academic papers → `raw/papers/`
     - Repository READMEs / technical docs → `raw/repos/`
     - Meeting / podcast transcripts → `raw/transcripts/`
     - Data files → `raw/data/`
   - Reuse the original filename as provided by the user (e.g. `foo.md`)
   - If a file with the same name already exists at the target path, **pause and ask the user**:
     ```
     raw/{subdir}/{filename} already exists. Choose:
     1. Overwrite the existing file
     2. Save as {filename-2}.md (increment suffix)
     ```
   - Write the file per the user's choice, then continue
   - Read `wiki/index.md` to get the current global Wiki view, then read `wiki/hot.md` for current focus

2. **Analyze content, extract 3–5 key points**
   - What is the core argument?
   - Which concepts are involved (need to create/update `wiki/concepts/` pages)?
   - Which entities are involved (people/companies/products — need to create/update `wiki/entities/` pages)?
   - What connections or contradictions exist with existing Wiki content?
   - Is this relevant to the current focus in `wiki/hot.md`?
   - **Comparisons check**: Does the new source have **≥3 comparable dimensions** of competition or substitution with an existing concept/tool (e.g. two frameworks, two methodologies)? If yes, create a `wiki/comparisons/` page in step 5a instead of cramming comparison content into concepts.

3. **Create `wiki/sources/summary-{slug}.md`**
   ```yaml
   ---
   title: "{Original title}"
   created: {today's date}
   modified: {today's date}
   tags: [#source, #{topic tag}]
   related: []
   summary: "{One-sentence summary}"
   source_path: "raw/..."
   ---
   ```
   Content includes: summary, list of key points, notable quotes, connections/contradictions with existing Wiki

4. **Update `wiki/index.md`**
   - Add entry to Sources section, update count
   - If new concepts exist, add entries to Concepts section, update count
   - If new entities exist, add entries to Entities section, update count
   - If new comparison pages exist, add entries to Comparisons section, update count

5. **Update relevant concept pages (`wiki/concepts/*.md`)**
   - Create if absent (with full frontmatter)
   - Append the new source's perspective at the end if it exists, update `modified` date

5a. **Create comparison page (if step 2 determined one is needed, `wiki/comparisons/*.md`)**
   - File naming: `{slug-a}-vs-{slug-b}.md`
   - Content structure: comparison dimensions table + per-dimension detail + use cases + recommendation
   - frontmatter tags include `#comparison`
   - Also add a link to this comparison page at the bottom of the relevant concepts pages

6. **Update relevant entity pages (`wiki/entities/*.md`)**
   - Same as step 5

7. **Flag contradictions (if any)**
   - Annotate with `> ⚠️ Contradiction: ...` in the summary page and in related concept/entity pages

8. **Append to `wiki/log.md`**
   ```
   ## [{date}] ingest | {filename}
   - Created: wiki/sources/summary-{slug}.md
   - Updated: wiki/index.md
   - New concepts: {list, or "none"}
   - Updated concepts: {list, or "none"}
   - New entities: {list, or "none"}
   - Updated entities: {list, or "none"}
   - New comparison pages: {list, or "none"}
   - Contradictions: {description, or "none"}
   ```

9. **Update `VAULT-INDEX.md`**
   - Increment Sources count by 1
   - Append this ingest to the recent activity list

10. **Update `wiki/hot.md`**
    - Append to the "Recent Activity" block:
      ```
      - {date} ingest | {source filename}: {1–2 sentence key points}, new pages: {list}
      ```
    - If hot.md exceeds 500 words, compress the oldest activity records (preserve structure, trim detail)

11. **Git sync**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "ingest: {filename}"
    git push
    ```

## Output

Report: which files were created/updated, the 3–5 key points discovered, whether any contradictions were found, whether Git push succeeded.
