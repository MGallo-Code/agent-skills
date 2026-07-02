# coding-mastermind - MANIFEST

The kit's provenance record: the build date and the tool-version baseline the kit was
built against, so `coding-mastermind-update` has a baseline to diff future platform
versions against. This is the system-level analog of a `package-lock.json`.

- **Kit version:** v1.0
- **Implementation date:** 2026-06-16
- **Last re-stamped:** 2026-07-01 (full update on WSL/Linux: Gemini CLI 0.46.0 -> 0.49.0;
  Claude Code confirmed already latest at 2.1.198; Codex re-pinned 0.141.0 -> 0.142.4 to
  match the installed/working binary (out-of-band drift; NOT floated to latest 0.142.5 - the
  config.toml MCP schema stays version-volatile). Toolchain on this WSL box: node v22.23.1,
  npm 10.9.8, git 2.34.1 - the macOS build-machine rows below are the original build record,
  left as-is. Prior re-stamp 2026-06-21 migrated Codex off the macOS Homebrew cask to npm.)
- **Built/verified on:** macOS (darwin). Note: `timeout` is absent on macOS; use
  `gtimeout` (coreutils) or the harness timeout. `gtimeout` was NOT installed at
  build time.

## Agent-platform baseline (disk-verified; built 2026-06-16, re-verified 2026-06-17)

| Tool | Baseline version | Notes |
|------|------------------|-------|
| Claude Code | 2.1.198 | latest on 2026-07-01 (`npm view @anthropic-ai/claude-code version`); npm global |
| Codex CLI | 0.142.4 | model GPT-5.5; npm global, PINNED (re-pinned 2026-07-01 to match installed; do NOT float to 0.142.5). `codex exec` sandbox default is version-volatile - always pass `-s read-only`; default not re-run empirically on 0.142.4 (mitigation makes it moot) |
| Gemini CLI | 0.49.0 | `gemini --approval-mode plan` is read-only plan mode (re-confirmed 2026-07-01 on 0.49.0 `--help`: `plan (read-only mode)`). Free Code Assist login ended 2026-06-18, metered after |
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
