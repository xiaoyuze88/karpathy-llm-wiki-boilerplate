---
name: lint-wiki
description: Use this skill when the user says "Lint the wiki", "lint wiki", "health check", "check the wiki for issues", "are there issues with the wiki", or "audit the wiki". Scans the entire wiki/ directory, detects contradictions, orphan pages, missing concepts, outdated statements, and structural issues, generates a diagnostic report with recommended next steps.
---

# lint-wiki

Scan the `wiki/` directory, detect quality issues, output a diagnostic report.

## Steps

1. **Scan all wiki/ files**
   - Read `wiki/index.md` for the global view and page list
   - Read all .md files in each subdirectory one by one

2. **Detect contradictions**
   - Find mutually contradictory descriptions of the same concept across different pages
   - Flag potentially outdated specific data/version numbers (created more than 6 months ago)

3. **Find orphan pages**
   - Pages not referenced by any other wiki/ page via `[[link]]` syntax

4. **Find missing concepts**
   - Terms/concepts mentioned in ≥3 pages but without a dedicated `wiki/concepts/` page

5. **Check structural integrity**
   - Do counts in `wiki/index.md` match actual file counts?
   - Are all page frontmatters complete (title/created/modified/tags/summary — all five required fields)?
   - Are entries in `wiki/log.md` consistently formatted (should be `## [YYYY-MM-DD] operation type | description`)?

6. **Suggest new sources**
   - Based on the existing Wiki's topic scope and gaps, recommend 5–10 sources worth adding
   - Each suggestion includes: topic + why it's valuable + possible search keywords

7. **Generate report and fix plan** (output to conversation)

   Report format:
   ```
   ## Lint Report [{date}]

   ### Contradictions ({n} found)
   - Page A vs Page B: {describe the contradiction}
     → Suggestion: which version to keep, or how to reconcile

   ### Orphan Pages ({n} found)
   - [[page path]] — {reason}
     → Suggestion: add a reference from which page, or delete

   ### Missing Concepts ({n} found)
   - "{concept}" — appears in: {page list}
     → Suggestion: create wiki/concepts/{slug}.md

   ### Structural Issues ({n} found)
   - {specific issue description}
     → Suggestion: {how to fix}

   ### Outdated Statements ({n} found)
   - Page: {path}, content: {original text}, created: {date}
     → Suggestion: update or mark as outdated

   ### Suggested New Sources ({n})
   - {topic}: {rationale} [search terms: {keywords}]
   ```

   After generating the report, list fixes by priority:
   ```
   ## Fix Plan

   P0 [Fix now]      {issue} → {specific action}
   P1 [Recommended]  {issue} → {specific action}
   P2 [Optional]     {issue} → {specific action}
   ```

   **After outputting the fix plan, wait for user confirmation. Once confirmed, execute fixes in priority order (edit wiki pages directly).**

8. **Execute fixes (after user confirmation)**
   - Fix items in P0 → P1 → P2 order
   - Brief the user after each fix: "Fixed: {issue}"
   - Proceed to next step when all done

9. **Append to `wiki/log.md`**
   ```
   ## [{date}] lint | health check
   - Contradictions: {n} found ({m} fixed)
   - Orphan pages: {n} found ({m} fixed)
   - Missing concepts: {n} found ({m} fixed)
   - Structural issues: {n} found ({m} fixed)
   - Outdated statements: {n} found ({m} fixed)
   ```

10. **Update `VAULT-INDEX.md`**
    - Append this lint to recent activity

11. **Update `wiki/hot.md`**
    - Append to "Recent Activity":
      ```
      - {date} lint | found {n} issues, fixed {m}, pending: {unfixed issue list}
      ```
    - If hot.md exceeds 500 words, compress the oldest activity records

12. **Git sync**
    ```bash
    cd ~/knowledge-vault
    git add .
    git commit -m "lint: health check {date}"
    git push
    ```

## Output

Lint report + fix plan (wait for user confirmation) → execute fixes → report fix results + Git push status.
