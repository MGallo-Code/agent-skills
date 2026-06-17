---
name: coding-mastermind-research-refresh
description: Re-runs the AI-coding-systems research sweep into the knowledge wiki - re-verifies the ~20 "AI Coding Systems" pages for staleness (dead URL, a version slip a reader would quote, a verdict the newest source moved), updates pages in place, and writes a dated research-delta. Reuses the existing research/refresh skill patterns; does not invent a new harness. Use when the AI-coding research may have moved or on a periodic knowledge refresh.
---

# coding-mastermind-research-refresh

## Overview

The kit's design rests on a body of research (the failure patterns, the
enforcers-vs-advisors evidence, the technique verdicts) that lives in the knowledge
wiki's "AI Coding Systems" section. That research goes stale: a cited tool ships a
breaking change, a URL dies, a newer paper moves a verdict. This command is the reusable
form of the original research sweep - it re-verifies those pages and records what moved.
It refreshes KNOWLEDGE (the wiki); its sibling `coding-mastermind-update` refreshes the
TOOLING baseline.

It reuses the existing research and refresh skill patterns in the wiki's own `.claude`;
it does NOT invent a new research harness (that would be the sprawl the kit fights). All
wiki writes go through ONE serialized writer (never two agents on one page).

## When to Use

- Periodically, to keep the AI-coding-systems knowledge current.
- After a notable platform/research event (a major agent release, a widely-cited new
  paper) that may have moved a verdict the kit cites.

**When NOT to use:** to update per-repo invariants (that is the repo's `INVARIANTS.md`)
or the tooling baseline (that is `coding-mastermind-update`).

## Process

1. **Baseline.** Find the last refresh date (the wiki's log) and the page set for the
   "AI Coding Systems" section (the wiki index routes them).
2. **Re-verify each page** for staleness, where stale = (a) a dead/redirected URL, (b) a
   version or number a reader would quote that has slipped, or (c) a verdict the newest
   source has moved. Use read-only research (web fetch/search) for diagnosis; fan-out is
   allowed here because it is read-only.
3. **Update pages in place** through ONE serialized writer, smallest-correct-change: fix
   the dead link, correct the number, update the verdict with the new citation. Do not
   reword for style or restructure ("don't drift the docs").
4. **Write a dated research-delta** summarizing what changed (the reusable form of the
   research handoff), so the next refresh and the kit's maintainers see the diff. Use a
   DISTINCT dated filename and never overwrite an existing file (read before write): a
   delta is a NEW artifact, not an edit of a prior one - reusing a prior delta's or a seed
   research handoff's name silently clobbers it (the exact stale/lost-doc failure the kit
   exists to prevent).
5. **Reconcile with the kit.** If a moved verdict changes an ADR or a MANIFEST capability
   fact, flag it for `coding-mastermind-update` rather than editing kit files here.

## Common Rationalizations (reject these)

- "Rewrite the page while I'm here." No - smallest correct change. Fix the stale fact,
  leave the rest.
- "Spin up a new research workflow." Reuse the existing research/refresh patterns; a new
  harness is sprawl.
- "Two agents can split the pages." Not on shared wiki pages - one serialized writer, or
  you get two edits racing the same file.

## Red Flags

- A new bespoke research harness instead of the existing pattern.
- Parallel writers on the wiki.
- Page rewrites that go beyond the stale fact (style/structure churn).
- A moved verdict applied to kit files directly (route it through update).

## Verification

- Each "AI Coding Systems" page was checked; stale facts were corrected with a citation.
- A dated research-delta exists and is non-empty.
- All wiki edits went through one serialized writer; no page was rewritten for style.

## Related

- `coding-mastermind-update` - the tooling-baseline sibling.
- The wiki's own research/refresh skills (the patterns this reuses).
