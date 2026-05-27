---
name: link-pages
tier: universal
description: Scans pages across all Notion HQs (except TrustLayer) and adds "Related Pages" sections for Wikipedia-style contextual navigation. Run periodically to improve cross-HQ discoverability. Triggers on "link pages", "connect my notes", "add related pages", "wiki links", "contextual navigation", "link up my Notion", or "connect my documents".
argument-hint: "[optional: HQ name to scope, e.g. 'Icon', 'PoaM', 'majorGTM', 'JR OS', or leave blank for all HQs]"
---

# Link Pages — Wikipedia-Style Contextual Navigation

Traverse all non-TrustLayer HQs, find semantic connections between pages, and add "Related Pages" sections to enable Wikipedia-style navigation through Jason's Notion workspace.

## Philosophy

Every document should be a node in a knowledge graph, not an island. When "Alfie Architecture" mentions "Project Registry," that page should surface a link back to it. When "Icon Product Vision" talks about "Style DNA," the reader should be able to jump directly to that feature doc.

This skill builds that web of connections — non-destructively, by appending a "Related Pages" section to each content page that has meaningful connections elsewhere in the workspace.

---

## HQs in Scope

Load these from `~/.claude/memory/project-registry.md` and `~/.claude/memory/hq-structures.md`:

| HQ | Page ID |
|---|---|
| JR OS (8thday.io) | `78d0d32a-2816-82ac-bb6c-8102dd589ee1` |
| Icon | `31e0d32a-2816-8114-824a-d7208341ee85` |
| majorGTM | `2590d32a-2816-8033-85cb-cf9a2d964df1` |
| PoaM | `2d50d32a-2816-812b-8855-dc198be78feb` |
| Radhouse Games | `3080d32a-2816-81ba-88cc-e9a152a56eff` |

> **NEVER touch TrustLayer.** If a page's ancestor path leads to the TrustLayer workspace, skip it unconditionally.

---

## Step 0: Handle Arguments

- If an argument is provided and matches an HQ name (case-insensitive) → scope to that HQ only
- If no argument → process all HQs listed above
- Announce scope before starting: "Linking pages across: [HQ list]"

---

## Step 1: Build the Page Index

For each HQ in scope, build a flat list of all linkable pages.

**How to traverse:**
1. Fetch the HQ root page
2. Extract all `<page url="...">` children (not `<database>` entries — skip those)
3. For project app pages (pages that contain more child pages, like "Alfie OS", "Icon app page"): fetch those too and extract their children (go 2 levels deep, not more)
4. Flatten everything into a single list

**Each entry in the index:**
```
{ title: "Product Vision", url: "https://www.notion.so/...", hq: "JR OS" }
```

**Skip from index:**
- Database pages (inline or standalone) — they're rows, not documents
- The HQ root pages themselves (too generic to link to meaningfully)
- Pages whose title is blank or generic (e.g., untitled pages)

---

## Step 2: Process Pages

Work through the index in batches of 10 pages. For each page:

### 2a. Fetch Content

Use `mcp__notion-personal__notion-fetch` with the page URL.

**Skip if:**
- Page content (stripped of whitespace and markup) is under 150 characters → it's a stub
- Page content contains only a database embed and no prose
- Page is an HQ root or navigation page

### 2b. Check for Existing Related Pages Section

Search the content for `## Related Pages` or `## 🔗 Related Pages`.

- If found → note existing linked URLs, prepare to **merge** (add new links, keep old ones)
- If not found → prepare to **append** a new section

### 2c. Find Related Pages (run both strategies)

**Strategy A — Title Mention Scan:**
- For each page in the index, check if its title appears verbatim in the current page's content (case-insensitive)
- If yes → it's a strong candidate
- Skip: self, immediate parent page, pages already linked in the main body text

**Strategy B — Semantic Search:**
- Call `mcp__notion-personal__notion-search` with `query: "[current page title]"`, `query_type: "internal"`, `page_size: 8`
- Filter results: must be type `notion` (not `notion-calendar`, `notion-mail`, etc.), must not be self, must not be in TrustLayer
- Take up to 4 results that aren't already linked in this page's content

**Merge and rank:**
- Combine A and B candidates, deduplicate by URL
- Title mentions rank higher than pure semantic matches
- Remove pages already linked anywhere in the main content
- Cap at **5 related pages maximum** — quality over quantity

### 2d. Skip if No Candidates

If no candidates survive after filtering, skip this page. Don't add an empty or trivial Related Pages section.

---

## Step 3: Update Pages

For each page that has 1+ candidate related links:

### If no existing Related Pages section

Use `mcp__notion-personal__notion-update-page` with:
- `command: "update_content"`
- `content_updates`: find the last meaningful line of content and append the new section after it

```json
{
  "old_str": "[last meaningful line of existing content]",
  "new_str": "[last meaningful line]\n\n---\n\n## 🔗 Related Pages\n\n[bullet list]"
}
```

### If existing Related Pages section

Use `update_content` to replace just the Related Pages section:
```json
{
  "old_str": "## 🔗 Related Pages\n\n[existing content of that section]",
  "new_str": "## 🔗 Related Pages\n\n[merged: old links + new links]"
}
```

### Format for the Related Pages section

```markdown
---

## 🔗 Related Pages

- [Page Title](https://www.notion.so/page-id) — why it's related (one line)
- [Page Title](https://www.notion.so/page-id) — why it's related (one line)
```

The "why it's related" annotation is required — it makes the link useful instead of just a title dump. Keep it to 5–8 words: "sister feature that handles purchase flow", "the deployment guide for this system", "foundational framework this builds on."

---

## Step 4: Report

After processing all pages, output a clean summary:

```
📎 Link Pages — Complete

Scope: [HQ names processed]
Pages scanned: N
Pages updated: N
Links added: N total
Pages skipped (stub): N
Pages skipped (no matches): N
Errors: N

Updated:
  ✓ [Page Title] (HQ Name) → [Page A], [Page B]
  ✓ [Page Title] (HQ Name) → [Page C], [Page D]

Skipped:
  — [Page Title] — stub (< 150 chars)
  — [Page Title] — no meaningful matches found
```

If you notice a cluster of pages that are densely interconnected, call it out: "Found a tight cluster in Icon around the styling features — might be worth a dedicated index page."

---

## Safety Rules

1. **Never touch TrustLayer** — check ancestor path if unsure about a page's origin
2. **Only ADD content** — never remove existing text (use `update_content`, not `replace_content` unless the page is a confirmed stub)
3. **Preserve child pages and databases** — don't wipe them with replace_content
4. **Skip stubs** — < 150 chars of real content means nothing useful to link from
5. **Max 5 related links per page** — avoid link soup
6. **Dedup before writing** — don't add a link already present in the page body
7. **No self-links** — never link a page to itself
8. **If existing section has 5+ links** — skip unless a new candidate is clearly stronger than existing ones
9. **If update fails** → log the error, skip the page, continue (no retries)

---

## Edge Cases

- **Page fetch fails** → log and skip
- **Page content is only a database** → skip (no prose context to link from/to)
- **Parent-child pages** → skip direct parent/child relationships (too obvious to be useful)
- **HQ root pages** → skip (would accumulate too many generic links)
- **Databases rows** → skip (not narrative pages)
- **Pages with only 1 line of content** → skip

---

## Alfie Touch

This is connective tissue work. Each link added makes the workspace more like a brain and less like a filing cabinet. After running, tell Jason:

- How many connections were made across HQs (cross-HQ links are especially valuable)
- Whether any previously isolated pages (0 related pages before) now have connections
- Any patterns you noticed: topics that appear across multiple HQs, orphan pages worth filling out

Frame it as building the map, not just filing documents. "Your Icon and majorGTM pages are starting to reference each other — the customer journey docs especially. That's the connective tissue you want."
