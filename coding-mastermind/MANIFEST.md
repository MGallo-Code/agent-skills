# coding-mastermind - MANIFEST

The kit's provenance record: the build date and the tool-version baseline the kit was
built against, so `coding-mastermind-update` has a baseline to diff future platform
versions against. This is the system-level analog of a `package-lock.json`.

- **Kit version:** v1.0
- **Implementation date:** 2026-06-16
- **Last re-stamped:** 2026-06-21 (focused change: Codex migrated from the macOS-only
  Homebrew cask to npm `@openai/codex`, PINNED to 0.141.0, so one cross-platform channel
  manages it on macOS AND WSL - the cask could not be managed on WSL, which let Codex drift
  on a second machine and break MCP wiring. Claude/Gemini baselines below are unchanged from
  the 2026-06-17 re-verify; run a full `coding-mastermind-update` to re-stamp those.)
- **Built/verified on:** macOS (darwin). Note: `timeout` is absent on macOS; use
  `gtimeout` (coreutils) or the harness timeout. `gtimeout` was NOT installed at
  build time.

## Agent-platform baseline (disk-verified; built 2026-06-16, re-verified 2026-06-17)

| Tool | Baseline version | Notes |
|------|------------------|-------|
| Claude Code | 2.1.179 | unchanged since build (`npm view @anthropic-ai/claude-code version`) |
| Codex CLI | 0.140.0 | model GPT-5.5; Homebrew cask. `codex exec` now defaults to `--sandbox read-only` (was `workspace-write` on 0.139.0) - still pass `-s read-only` explicitly; the default is version-volatile |
| Gemini CLI | 0.46.0 | `gemini --skip-trust --approval-mode plan` is read-only plan mode (re-confirmed on 0.46.0 `--help`). Free Code Assist login ends 2026-06-18, metered after |
| node | 22.17.1 | the gate checks are plain ESM `.mjs` |
| npm | 11.11.0 | |
| git | 2.50.1 (Apple) | |

## Capability facts the kit depends on (re-check these on update)

- **PreToolUse exit-2** reliably blocks Bash in Claude Code 2.1.x; it does NOT reliably
  block Write/Edit or MCP. So the hard gate for Write-touching invariants is the CI
  check RE-RUN at the `Stop` gate, not a per-write PreToolUse hook.
- **Stop / SubagentStop** can attach `additionalContext` (since v2.1.151) and are
  capped at 8 blocks. A Stop hook blocks turn completion via exit 2 or
  `{"decision":"block","reason":...}`.
- **`.claude/skills` auto-load** (since v2.1.157); `disallowed-tools` skill frontmatter
  (since v2.1.152); output-styles were REMOVED (v2.1.91) - do not depend on them.
- **The portable enforcement layer is CI** (vendor-neutral GitHub Actions) plus the
  constitution/rules file every agent reads. Hooks are Claude-Code-specific. The
  invariant LIVES in CI; each agent gets the best in-loop enforcement its own harness
  supports (Codex has `/review` + an OS sandbox; Gemini has its own).
- **Cross-vendor CLI dispatch (the cross-check skill), re-verified 2026-06-17:** the
  `codex exec` sandbox default is VERSION-VOLATILE - `workspace-write` on 0.139.0,
  `read-only` on 0.140.0 (disk-verified via the `codex exec` session header, both in and
  out of a git repo). So ALWAYS pass `--sandbox read-only` (`-s read-only`) explicitly and
  never depend on the default. `gemini --approval-mode plan` remains read-only (0.46.0
  `--help` documents `plan (read-only mode)`). And a Claude Code Task/Workflow SUBAGENT's
  sandbox classifier blocks the vendor-CLI call as private-source exfiltration, so the
  dispatch must run in the main loop (sandbox disabled for that call) or behind an explicit
  `codex`/`gemini` Bash allowlist; the worker context-hygiene benefit is then traded off.
  Re-check both on update.

## Gate-tool baselines (recommended pins; install per-repo as needed)

These are the linters/scanners the kit's checks can use. None except node are required
for the SBIC instance (its checks are pure `.mjs` + pgTAP). Marked "not installed" were
not on PATH at build; treat their pins as fresh, unvalidated adoptions to verify before
relying on their output.

| Tool | Recommended pin | Installed at build? | Purpose |
|------|-----------------|---------------------|---------|
| ast-grep CLI + `ast-grep-mcp` | 0.43.0 (mcp 2026-06-13) | no | structural rule that is a CI gate AND agent-callable |
| jscpd | 5.0.9 (Rust engine; v4.2.5 = legacy TS) | no | clone/duplication detection (validate v5 output before trusting counts) |
| knip | 6.17.1 | no | dead-code / unused-export detection |
| dependency-cruiser | 17.4.3 (do NOT float to v18 beta) | no | import-graph rules |
| Squawk | 2.58.0 | no | migration linter |
| osv-scanner / socket | latest | no | hallucinated-package + advisory check (item 11b) |
| gitleaks | (already in SBIC CI) | n/a | secret scan |
| mutation testing | cargo-mutants (Rust) / Stryker (TS) / mutmut (Python) | no | diff-scoped surviving-mutant check |
| spec-kit | 0.10.2 | no | steal the constitution file + `/analyze` pass only |

## How to refresh this baseline

Run `coding-mastermind-update`: it reads this file, fetches current tool versions
(`npm view @anthropic-ai/claude-code version` and the Codex/Gemini equivalents +
their changelogs), diffs against the capability facts above, and proposes a
changes list + a re-stamped MANIFEST for approval. It PROPOSES, never auto-applies.
