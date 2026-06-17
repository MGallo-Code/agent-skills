# coding-mastermind - PLAN

Build order for the generic phase, highest-leverage first. Folds in the three plan
deltas (A: registry + in-loop interception primary, scheduled audit backstop; B:
cross-vendor adversarial cross-check command; C: a kit usage command). Status
tracked here as it lands.

## Canonical-homes map (state each fact's ONE home before writing it)

- System concepts + the "why": `~/Documents/Wiki/wiki/` (AI Coding Systems section).
  Interaction-matrix canonical home is here.
- Cross-repo rules (agent-agnostic): `~/Documents/EA/claude-config/global-rules/`.
- Reusable skills / hooks + the kit + MANIFEST + maintenance commands:
  `~/Documents/agent-skills/` (distributed by the rail).
- SBIC specifics (k-anon check, SBA-form grounding, the renumber fix, the SBIC
  INVARIANTS instance): the SBIC tree (`platform/`, `.claude/`, `docs/`).
- Charon specifics: the Charon repo.
- The delivery rail config: `~/.dotfiles/manifest.sh` + `sync.sh`.

## Write-serialization rule (convergent, few-agent)

- Route ALL writes to the SBIC `docs/` tree and the Wiki through a SINGLE
  serialized writer (one agent, in sequence, one context). Never two writers on one
  page.
- Fan-out is allowed ONLY for read-only diagnosis.
- Workflow agents never write files: an agent emits the literal contents, the
  orchestrating session writes them.

## Status legend

[x] done  [~] in progress  [ ] not started

## Slice 0 (DONE, pushed + PR'd 2026-06-16)

- [x] `platform/scripts/ci/check-cohort-floor-coverage.mjs` + allowlist + CI job
      (platform PR #235). k-anon COVERAGE enforcer. Local-green and CI-green.
- [x] renumber-rule reframed as default-with-audited-escape-hatch across 6
      `.claude/` files (docs PR #3) + `platform/scripts/ci/.migration-renumber-approvals.txt`.
- [x] Stop-gate hook re-running the coverage check in-loop, no-op outside SBIC
      (agent-skills PR #1).

## Generic phase

1. [~] Spec + Plan (item 5) - this file + SPEC.md.
2. [ ] Per-repo INVARIANTS registry (item 7 / Delta A.1): `platform/INVARIANTS.md`,
       surface-indexed, with recurrence_count / detection_signals / enforcer_status.
       k-anon = first entry. STOP for Michael's reaction after this.
3. [ ] In-loop interception (item 8 / Delta A.2): extend `codebase-locator` to emit
       applicable INVARIANTS rows + recurrence for touched files; draft the enforcer
       in-PR on recurrence / new unguarded surface. Precision-first signals. Plus:
       remove the stale `workos-server.ts` locator entry; add a link-integrity lint.
4. [ ] Kit skeleton + `coding-mastermind-init` (items 1,6,11): scaffolds a layered
       CLAUDE.md + INVARIANTS stub + DoD/PR template into any repo. Plus MANIFEST.md.
5. [ ] Cross-vendor cross-check command (Delta B): packages a claim/diff, dispatches
       to Codex + Gemini read-only/sandboxed prompting them to REFUTE, synthesizes a
       PROPOSAL. Single context-hygiene worker returns only the distilled verdict
       (preserving dissent). NOT a majority vote.
6. [ ] Scheduled BACKSTOP audit (Delta A.3) + `coding-mastermind-research-refresh` +
       `coding-mastermind-update` (item 13). Backstop must NOT depend on the
       cross-vendor CLIs (headless auth is unreliable). Each RUN once.
7. [ ] Cross-agent portability fix (items 14, 5a): diff-guard + `chmod 0444` in
       `manifest.sh` regen_combined_agent_rules; document the new-agent extension
       point; close the `platform/AGENTS.md` stub gap.
8. [ ] Charon generality proof (item 16) + sync-rail integration (item 15) + wiki
       updates (items 17-18, one serialized writer).
9. [ ] Kit USAGE command (Delta C), last - once there is a kit to describe.
10. [ ] Remaining items: critic charter + DoD/PR template + soft spec-gate (10);
        shared-repo PR standards in git.md (10b); SBA Form 1031 domain-grounding
        registry (11); hallucinated-dep + secret guard (11b); ADR decision-log home
        `SBIC/docs/decisions/` + WorkOS worked example + doc-drift fixes (12);
        completeness-critic pass (18).

## Guardrails for the build

- No commit/push/PR/merge without explicit per-action permission. Never self-merge.
- Branch off main; sync first; stage only changed files by explicit path.
- Disk wins. Re-ground every "X is/isn't wired" claim against disk.
- Prove it: a check counts as fixing CI only when GREEN on a real PR.
- Smallest correct change; extend existing skills/hooks/CI over new ones.
- No personal-tool references in team-cloned SBIC repos.
