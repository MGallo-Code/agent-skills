---
name: update-coding-agents
description: Upgrades the installed coding-agent CLIs (Claude Code, Gemini CLI, Codex, and any others) to their latest versions, each via its REAL install channel (npm global / Homebrew cask / native updater), reports before->after versions, then re-stamps the kit baseline. Use when the user says "update/upgrade my agents", "update claude / codex / gemini", or on a periodic tooling refresh. Distinct from coding-mastermind-update, which only PROPOSES MANIFEST-baseline changes and never installs anything.
---

# update-coding-agents

## Overview

Upgrades the agent CLIs THEMSELVES (the binaries) to latest, each through the channel it
was actually installed with. This is the ACTOR that installs; its sibling
`coding-mastermind-update` is the PROPOSER that only re-stamps the kit's version baseline.
Run this to upgrade, then run that to record the new baseline.

The one rule that makes this safe: **upgrade each tool via the channel it was installed
with.** npm-installing a Homebrew tool (or vice versa) leaves two copies on `PATH` and the
wrong one wins. So this skill DETECTS the channel per tool, never assumes.

## When to Use

- "Update / upgrade my agents", "update claude / gemini / codex", "update my coding tools".
- A periodic tooling refresh, or after a notable agent release.

**When NOT to use:** to change the kit's recorded baseline only (that is
`coding-mastermind-update`, propose-only); mid-critical-session if you depend on the
current Claude Code build (an upgrade takes effect on the NEXT launch, so finish first);
when versions are deliberately pinned (then bump the pin, do not float to latest).

## Process

1. **Inventory + detect channel** for each agent. Record the current version and HOW it is
   installed (the upgrade command depends on it):
   - **npm global:** `npm ls -g --depth=0 <pkg>` resolves -> upgrade with
     `npm install -g <pkg>@latest`.
   - **Homebrew cask:** `which <bin>` resolves into `…/Caskroom/<name>/…` ->
     `brew upgrade --cask <name>`. (A cask is NOT a formula: `brew list <name>` and
     `brew upgrade <name>` miss it; you must say `--cask`.)
   - **Homebrew formula:** resolves into `…/Cellar/<name>/…` -> `brew upgrade <name>`.
   - **Native self-updater** (Claude Code installed via the native installer exposes
     `claude update`) -> use the tool's own updater.
2. **Show the plan BEFORE installing.** For each tool print `current -> latest`
   (`npm view <pkg> version`; `brew outdated --cask`/`--formula`). Do not upgrade silently:
   a major bump can move a capability fact the kit depends on (step 5).
3. **Upgrade each via its detected channel.** Known agents (verified on this machine
   2026-06-16; re-detect, do not trust this list blindly):
   | Agent | Channel | Upgrade command |
   |-------|---------|-----------------|
   | Claude Code (`@anthropic-ai/claude-code`) | npm global | `npm install -g @anthropic-ai/claude-code@latest` |
   | Gemini CLI (`@google/gemini-cli`) | npm global | `npm install -g @google/gemini-cli@latest` |
   | Codex (`codex`) | Homebrew **cask** | `brew upgrade --cask codex` |
   To add another agent ("etc"): detect its channel in step 1 and use the matching command.
   Tip: `brew update` once before the brew upgrades so cask metadata is fresh.
4. **Verify after.** Re-run each `--version`; emit a `before -> after` table. A tool already
   at latest is a no-op, report it as such (not a failure).
5. **Re-stamp the kit + re-verify capability facts.** Run `coding-mastermind-update`: it
   re-stamps `MANIFEST.md` to the new versions and re-checks the facts the kit RELIES on,
   which an upgrade can silently change - e.g. does `codex exec` still default to
   `--sandbox workspace-write` (so the cross-check still needs `--sandbox read-only`)? does
   `gemini --approval-mode plan` still mean read-only? did a Claude Code hook event get
   renamed/removed? Propose, do not auto-apply.

## Common Rationalizations (reject these)

- "Just `npm install -g` everything." No - Codex is a Homebrew cask here; npm-installing it
  gives you two copies and the wrong one shadows. Use the detected channel.
- "Upgrade them all silently, latest is latest." No - show `current -> latest` first; a
  version bump is exactly when a sandbox flag, approval mode, or hook event changes under
  you (that is why step 5 exists).
- "Skip the MANIFEST re-stamp." Then the kit's baseline drifts from reality and
  `coding-mastermind-update` has nothing true to diff against next time.

## Red Flags

- An upgrade run that does not record the version delta (before->after).
- `brew upgrade codex` WITHOUT `--cask` (silently does nothing for a cask, looks like
  success).
- Floating a deliberately-pinned tool to latest with no note.
- A capability-fact change (sandbox default, approval mode, hook rename) noticed but not
  routed into `coding-mastermind-update` / the MANIFEST.

## Verification

- Each agent reports a version >= its pre-run version (or "already latest"); none left
  half-upgraded.
- Every tool was upgraded via its detected channel (no npm/brew cross-contamination).
- `coding-mastermind-update` was run and the capability facts re-confirmed (or the changes
  proposed) on the new versions.

## Related

- `coding-mastermind-update` - the propose-only sibling that re-stamps the MANIFEST baseline
  (run it right after this).
- `coding-mastermind-cross-check` - depends on the codex/gemini sandbox + approval-mode facts
  this skill's step 5 re-verifies.
