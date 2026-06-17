---
name: coding-mastermind-cross-check
description: Runs a rare, high-stakes decision or diff past other-vendor flagship CLIs (Codex, Gemini) in read-only/sandboxed mode, prompting them to REFUTE it, then synthesizes one PROPOSAL that preserves the disagreement. Adversarial cross-check, NOT a majority vote. Use only for high-stakes or large-diff or strategic decisions; skip on Q&A and small edits.
---

# coding-mastermind-cross-check

## Overview

A second opinion is most valuable when it DISAGREES, so this gets one on purpose. It
packages a concrete claim, decision, or diff and asks other-vendor models to refute it,
not to praise it (which avoids the sycophancy that makes "what do you think?" useless).
Cross-vendor matters: a model is blind to its own failure modes, and a different vendor
surfaces bugs single-model review misses (Arbiter, arXiv:2603.08993, found a real
Gemini-CLI memory bug for $0.27 that single-model missed). The panel also seats a
FRESH-CONTEXT Claude (the dispatch worker, given the same packaged prompt): the main loop that
WROTE the claim is its worst judge, so a fresh Claude strips that author bias. It is a peer to
the two vendors, not a replacement - cross-vendor stays the PRIMARY diversity (a same-vendor
voice shares some blind spots); the fresh Claude's job is removing the author's bias.

This is the ONE multi-model technique the kit keeps. It explicitly rejects the others:
weight-merging (impossible on hosted models), mixed-model MoA (Self-MoA wins), and
output-voting/ensembling (the "popularity trap" - consensus is not correctness). So do
NOT turn this into a majority vote. The output is a synthesis that PRESERVES the
dissent, never a tally.

## When to Use

- A rare, high-stakes decision (an architecture bet, an auth change, an irreversible
  migration) or a large/strategic diff.
- A claim you want adversarially stress-tested before committing.

**When NOT to use:** Q&A, small edits, anything where one lever already suffices (pick
ONE lever per change-size). The CLIs cost their vendors' tokens and ~30-120s each, so
this is on-demand and interactive only. The scheduled backstop audit must NOT depend on
it (headless auth is unreliable).

## Process

1. **Frame the claim to refute.** Write a tight, self-contained prompt: the exact claim
   or decision, the minimal context to judge it (paste the diff or the key code, do not
   assume the model has the repo), and the instruction: "Try to REFUTE this. Find the
   strongest case that it is wrong, unsafe, or will not work. If you cannot, say so and
   why." Ask for a verdict + the single strongest counter-argument, not an essay.
2. **Delegate the dispatch to ONE context-hygiene worker.** The raw multi-model output
   (each model's full answer + CLI metadata) is large and would bloat the rest of this
   session and get re-read every turn. So spawn a SINGLE subagent (Boomerang
   context-hygiene delegation: one worker, motive = context-rot, not parallelism - it
   does NOT violate the convergent/few-agent rule) and pass it the framed prompt
   EXPLICITLY (a fresh worker, not a context-inheriting fork, unless the decision truly
   needs the full conversation). Because the worker is a FRESH-CONTEXT Claude (the fresh worker above, not a
   context-inheriting fork), it is also an INDEPENDENT refuting voice the author lacks: have it
   (a) refute the claim ITSELF from that fresh context - stripping the bias of the main loop
   that wrote the claim - THEN (b) run the vendor CLIs. It is now a refuter on the panel, not
   just a dispatcher, so use a CAPABLE model; the intelligence is in all three replies, not
   only the vendors'.
3. **The worker runs the other-vendor CLIs, read-only/sandboxed.** Never `--yolo` /
   `danger-full-access`. macOS has no `timeout`; use `gtimeout` or the harness timeout.
   ```bash
   codex exec --skip-git-repo-check --sandbox read-only "<refute prompt>" < /dev/null  # GPT-5.5; pass -s read-only explicitly - the exec sandbox default is version-volatile
   gemini --skip-trust --approval-mode plan -p "<refute prompt>" < /dev/null            # read-only plan mode
   ```
   The CLIs reach their own model API; that egress is the point. Two harness gotchas:
   (a) the `codex exec` sandbox default is version-volatile - `workspace-write` on 0.139.0,
   `read-only` on 0.140.0 (disk-verified 2026-06-17) - so ALWAYS pass `--sandbox read-only`
   (or `-s read-only`) explicitly and never depend on the default. (b) In Claude Code a
   Task/Workflow SUBAGENT's sandbox
   classifier blocks the vendor-CLI call as private-source exfiltration, so the dispatch
   often has to run in the MAIN loop (Bash sandbox disabled for that call) or behind an
   explicit `codex`/`gemini` Bash allowlist; running it in the main loop trades away the
   worker context-hygiene benefit, so distill the raw output yourself before continuing.
   If a CLI is unauthenticated or errors, report that vendor as "unavailable" rather than
   failing the whole pass (the fresh-context Claude refutation still stands); NEVER fabricate a
   vendor response.
4. **The worker returns ONLY the distilled verdict:** the key agreements, the
   DISAGREEMENTS verbatim-enough to be actionable (do NOT over-compress away the dissent
   - it is the whole point), and each refuter's strongest counter-argument (the fresh-context
   Claude, Codex, and Gemini). Not the raw transcripts.
5. **You synthesize a PROPOSAL.** Combine the THREE independent refutations (fresh-Claude +
   Codex + Gemini) into: (a) the claim's strongest surviving objection, (b) whether it changes
   the decision, (c) a recommended action. Your own (author) view is the claim UNDER TEST, not
   a fourth refuting vote - do not let it overrule the panel. Adversarial/diversity synthesis,
   never a vote count. PROPOSE; never auto-apply.

## Common Rationalizations (reject these)

- "Three models agreed, so it's right." Agreement is not evidence - judges correlate
  ("Nine Judges, Two Effective Votes," arXiv:2605.29800). Weight the strongest argument,
  not the headcount.
- "Ask them what they think." Sycophancy makes that worthless. Ask them to REFUTE.
- "Run it on every change." No - it is for rare high-stakes diffs. Routine use is noise
  and cost.
- "Let the worker keep the full transcripts in context." That is the context-rot this
  delegation exists to prevent. The worker returns the distilled verdict only.

## Red Flags

- A majority-vote or averaged score in the output. That is the rejected technique.
- The dissent compressed to "they mostly agreed." Surface the actual disagreement.
- Any CLI invoked with write/network access. Read-only/sandboxed only.
- The proposal auto-applied. It proposes; the human (or you, after the human's nod)
  decides.

## Verification

- The output names each vendor's verdict + strongest counter-argument, and a synthesized
  recommendation that preserves any disagreement.
- The raw transcripts did NOT enter the main context (only the distilled verdict did).
- No CLI ran with write/network/danger access.
- It PROPOSED; nothing was applied.

## Related

- The decision record this often feeds: `docs/decisions/` (ADR before an architecture bet).
- `coding-mastermind-help` - when to reach for this vs other levers.
- Rationale: kit `SPEC.md` (the enforcers-vs-advisors and multi-model verdicts).
