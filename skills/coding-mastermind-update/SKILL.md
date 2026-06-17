---
name: coding-mastermind-update
description: Updates the coding-mastermind kit itself by folding in new agent-platform capabilities - reads the MANIFEST baseline, fetches current tool versions and changelogs, diffs against the capability facts the kit depends on, and proposes a changes list plus a re-stamped MANIFEST. Proposes, never auto-applies. Use when agent platforms (Claude Code / Codex / Gemini) or the gate tools have moved and the kit may be stale.
---

# coding-mastermind-update

## Overview

Agent platforms drift: a hook event changes, a skill-frontmatter capability appears, a
sandbox flag is renamed, a primitive the kit depends on is removed (output-styles were
removed in Claude Code v2.1.91). When that happens silently, the kit's assumptions rot -
the system-level version of pattern #2/#6. This command is the named owner of that drift:
it diffs the live tool world against the recorded `MANIFEST.md` baseline and proposes
what to change. It PROPOSES, never auto-applies (smallest-correct-change; a human decides).

It is also the sanctioned path to revisit an ADR-2 SKIP verdict: if a platform change
makes a previously-skipped technique viable, that surfaces here.

## When to Use

- A new Claude Code / Codex / Gemini release, or you suspect the kit's baseline is stale.
- Onboarding a new agent platform (does the kit's portability story still hold?).
- Before relying on a capability fact the kit assumes (re-verify it is still true).

**When NOT to use:** routine per-repo work. This updates the SYSTEM, not a feature.

## Process

1. **Read the baseline.** Open `coding-mastermind/MANIFEST.md`: the recorded tool
   versions and the capability-facts list.
2. **Fetch current versions.** For each tool, get the latest:
   ```bash
   npm view @anthropic-ai/claude-code version          # Claude Code
   codex --version ; gemini --version                  # the other CLIs
   node --version ; ast-grep --version 2>/dev/null     # gate tools (may be absent)
   ```
   And read the changelog / GitHub releases for anything between the baseline and now.
3. **Diff against the capability facts.** "Relevant" = anything that changes a hook event,
   a skill-frontmatter capability, a sandbox flag, or DEPRECATES a primitive the kit uses.
   A pure version bump with no capability change is noted, not actioned.
4. **Re-verify the load-bearing facts.** Confirm the facts the kit's design rests on are
   still true (PreToolUse exit-2 reliable for Bash only; Stop/SubagentStop
   `additionalContext` + 8-block cap; `.claude/skills` auto-load; the CLI flags the
   cross-check skill uses). A fact that flipped is a required change.
5. **Output a proposed-changes list + a re-stamped MANIFEST.** For each relevant change:
   what moved, which kit file it affects, and the proposed edit. Plus the new MANIFEST
   with updated versions and date. Present for approval; do NOT apply.

## Common Rationalizations (reject these)

- "Just bump the version numbers." A version bump without checking capability changes
  misses the deprecation that breaks the kit. Diff the behavior, not just the string.
- "Apply the changes since I'm here." No - propose. The human owns the change to the
  system, exactly as with any architecture decision.
- "The fact is probably still true." Re-verify the load-bearing ones; a silently-flipped
  capability is how the kit rots.

## Red Flags

- Auto-applying edits to kit files or the MANIFEST.
- A "no changes" result without having actually read the changelogs (confirm you looked).
- Treating a version string change as the deliverable; the deliverable is the capability
  diff.

## Verification

- The output is a concrete changes list (each tied to a kit file + a proposed edit) plus
  a re-stamped MANIFEST, both non-empty when versions moved.
- The load-bearing capability facts were each re-checked and marked still-true or changed.
- Nothing was applied.

## Related

- The baseline it diffs: `coding-mastermind/MANIFEST.md`.
- `coding-mastermind-research-refresh` - refreshes the KNOWLEDGE (wiki); this refreshes
  the TOOLING baseline.
