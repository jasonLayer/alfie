# Alfie OS Curation Practice

Alfie OS is not a fork of gstack — it's a separate product that monitors gstack as a reference. This document describes the standing practice for staying informed about gstack changes and deciding what to incorporate.

## The Roles

**Alfie's role:** Surface. Review gstack's commit stream, summarize what shipped, and flag anything that looks relevant to Alfie OS.

**Jason's role:** Decide. Look at what Alfie surfaced, read the actual diff if curious, and say yes/no/build-better.

Alfie never auto-pulls gstack changes into Alfie OS. Every incorporation is a deliberate decision.

---

## When to Check

Check gstack at the start of a new project (before `/brainstorm`) or monthly — whichever comes first.

Don't check gstack reactively in the middle of a build. Surfacing gstack updates is a planning activity, not an execution one.

---

## How to Check

### See what gstack shipped since your last sync

```bash
git -C ~/.claude/skills/gstack fetch origin
git -C ~/.claude/skills/gstack log --oneline HEAD..origin/main
```

This shows commits gstack has on `main` that your local install hasn't pulled yet.

### See the full diff

```bash
git -C ~/.claude/skills/gstack diff HEAD origin/main
```

Or for a specific skill:

```bash
git -C ~/.claude/skills/gstack diff HEAD origin/main -- ship/SKILL.md
```

### Update your local gstack (optional)

```bash
cd ~/.claude/skills/gstack && git pull && ./setup --quiet
```

This updates gstack itself. It does NOT automatically change any Alfie OS skill.

---

## What Alfie Surfaces

When checking gstack, Alfie reads the commit log and diff and summarizes:

1. **New skills added** — does gstack now do something Alfie doesn't?
2. **Skill improvements** — did an existing gstack skill get meaningfully better?
3. **Framework changes** — did the compound-engineering engine change in ways that affect how Alfie's brainstorm/plan wrappers delegate?
4. **Bug fixes** — did gstack fix something that Alfie OS has the same issue with?

Alfie presents these as a short list: skill name, what changed, why it might matter for Alfie OS.

---

## How Jason Decides

For each item Alfie surfaces, the options are:

- **Bring it in** — copy the relevant section of the gstack SKILL.md into the Alfie OS skill, commit to `os-structure/skills/`
- **Skip it** — gstack's approach doesn't fit Alfie's use case or voice
- **Build better** — Alfie OS will build a native version that fits better (put it in `docs/plans/`)

When incorporating, always adapt to Alfie's voice — don't paste gstack content verbatim.

---

## What NOT to Pull From gstack

Some gstack internals are gstack-specific and should never be ported:

- **CE engine internals** — `compound-engineering:ce-*` is gstack's plugin marketplace. Alfie uses it as a black box via skill delegation. Don't copy CE engine skills into Alfie OS.
- **Browse binary** — the browser CLI and Playwright infrastructure. Alfie calls the gstack binary, not a copy of it.
- **Host configs** — `hosts/` is gstack's per-agent adapter layer. Not relevant to Alfie OS.
- **gstack-specific skills** — `benchmark`, `canary`, `gstack-upgrade`, `office-hours`, `retro` are gstack products. Alfie has its own equivalents or doesn't need them.
- **Build infrastructure** — `scripts/`, `test/`, `.github/` are gstack's dev tooling. Not for Alfie OS.

---

## Example Curation Session

```
Alfie: Checked gstack. 3 things since last review:

1. ship/SKILL.md — added automatic VERSION bump detection.
   → Could improve Alfie's /done skill (currently manual).
   → Your call: bring it in, adapt to Alfie voice?

2. investigate/SKILL.md — added a "timeline reconstruction" phase.
   → Alfie's /investigate already has 4 phases. This would make 5.
   → Might be worth it for complex bugs. Or too much ceremony?

3. review/SKILL.md — new P0 category for security issues.
   → Alfie's /review doesn't have P0. Could be useful.

Jason: Bring in #1 and #3. Skip #2 — four phases is enough.

Alfie: [reads gstack ship/SKILL.md, adapts VERSION bump section, 
        commits to os-structure/skills/done/SKILL.md]
        [reads gstack review/SKILL.md, adapts P0 category,
        commits to os-structure/skills/receive-review/SKILL.md]
```

---

## Keeping Track

After incorporating a gstack change, note it in a commit message:
```
feat(investigate): add timeline reconstruction phase (from gstack v1.42)
```

No need for a formal log — the git history is the record.
