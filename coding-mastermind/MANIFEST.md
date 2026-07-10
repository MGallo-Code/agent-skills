# coding-mastermind - MANIFEST

The kit's provenance record: the build date and the tool-version baseline the kit was
built against, so `coding-mastermind-update` has a baseline to diff future platform
versions against. This is the system-level analog of a `package-lock.json`.

- **Kit version:** v1.0
- **Implementation date:** 2026-06-16
- **Last re-stamped:** 2026-07-10 (WSL/Linux update: Codex pin DELIBERATELY BUMPED
  0.142.4 -> 0.144.1 after a full sandbox preflight (CODEX_HOME clone + npx candidate:
  forward/backward config parse both green, sync `mcp add` wiring schema byte-identical
  with no `transport` field, exec/mcp-add flags unchanged, `-s read-only` honored,
  PreToolUse hook trust survived the live upgrade). The pin now has a canonical
  machine-readable home - dotfiles `manifest.sh CODEX_PIN` / `manifest.ps1 $CodexPin` -
  enforced by a warn-on-drift check in both syncs (parity-gated) and bumpable via
  `~/.dotfiles/scripts/codex-pin-preflight.sh <version>`. macOS + the Windows PC must
  install 0.144.1 at their next sync (lockstep). Also: Gemini CLI 0.49.0 -> 0.50.0;
  Claude Code found already at npm latest 2.1.206. Verified `codex exec` default
  sandbox on this WSL machine is `danger-full-access` (on 0.142.4 AND 0.144.1), so
  cross-check must keep passing `-s read-only`. Toolchain this machine: WSL2,
  node v22.23.1, npm 10.9.8, git 2.34.1. Prior re-stamp 2026-07-02 covered macOS;
  2026-07-01 re-pinned Codex on WSL after out-of-band drift; 2026-06-21 migrated
  Codex off the macOS Homebrew cask to npm.)
- **Built/verified on:** macOS (darwin). Note: `timeout` is absent on macOS; use
  `gtimeout` (coreutils) or the harness timeout. `gtimeout` was NOT installed at
  build time.

## Agent-platform baseline (disk-verified; built 2026-06-16, re-verified 2026-06-17)

| Tool | Baseline version | Notes |
|------|------------------|-------|
| Claude Code | 2.1.206 | latest on 2026-07-10 (`npm view @anthropic-ai/claude-code version`); npm global (macOS: under nvm only, after removing the duplicate Homebrew-prefix install) |
| Codex CLI | 0.144.1 | model GPT-5.5; npm global, PINNED - canonical pin lives in dotfiles `manifest.sh CODEX_PIN` (+ `manifest.ps1 $CodexPin`), both syncs warn on drift; bump ONLY via `~/.dotfiles/scripts/codex-pin-preflight.sh <version>` (pin bumped 2026-07-10 from 0.142.4 after preflight PASS). `codex exec` sandbox default is version-volatile - `danger-full-access` on 2026-07-02 macOS 0.142.4 and 2026-07-10 WSL 0.142.4/0.144.1; always pass `-s read-only` |
| Gemini CLI | 0.50.0 | latest on 2026-07-10 (`npm view @google/gemini-cli version`); `gemini --approval-mode plan` is read-only plan mode for cross-check verification calls (re-confirmed 2026-07-10 on 0.50.0 `--help`: `plan (read-only mode)`). Free Code Assist login ended 2026-06-18, metered after |
| node | 22.17.1 | the gate checks are plain ESM `.mjs` |
| npm | 11.11.0 | |
| git | 2.50.1 (Apple) | |

## Capability facts the kit depends on (re-check these on update)

- **PreToolUse exit-2** reliably blocks Bash in Claude Code 2.1.x; it does NOT reliably
  block Write/Edit or MCP. So the hard gate for Write-touching invariants is the CI
  check RE-RUN at the `Stop` gate, not a per-write PreToolUse hook.
- **Stop / SubagentStop** can attach `additionalContext` (since v2.1.151). Current
  docs cap hook output strings at 10,000 characters and Stop continuation loops at
  8 consecutive continuations. A Stop hook blocks turn completion via exit 2 or
  `{"decision":"block","reason":...}`.
- **`.claude/skills` auto-load** (since v2.1.157); `disallowed-tools` skill frontmatter
  (since v2.1.152); output-styles were REMOVED (v2.1.91) - do not depend on them.
- **The portable enforcement layer is CI** (vendor-neutral GitHub Actions) plus the
  constitution/rules file every agent reads. Hooks are Claude-Code-specific. The
  invariant LIVES in CI; each agent gets the best in-loop enforcement its own harness
  supports (Codex has `/review` + an OS sandbox; Gemini has its own).
- **Cross-vendor CLI dispatch (the cross-check skill), re-verified 2026-07-10:** the
  `codex exec` sandbox default is VERSION-VOLATILE - `workspace-write` on 0.139.0,
  `read-only` on 0.140.0, and `danger-full-access` on 0.142.4 macOS (2026-07-02) and
  on 0.142.4/0.144.1 WSL (2026-07-10; all disk-verified via the `codex exec` session
  header). So ALWAYS pass
  `--sandbox read-only` (`-s read-only`) explicitly and never depend on the default.
  `gemini --approval-mode plan` remains read-only for cross-check verification calls
  (0.50.0 `--help` documents `plan (read-only mode)`). And a Claude Code Task/Workflow SUBAGENT's
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

Codex pin bumps additionally go through `~/.dotfiles/scripts/codex-pin-preflight.sh
<version>` (sandboxed forward/backward config-compat proof against a CODEX_HOME clone)
BEFORE the pin moves in dotfiles `manifest.sh`/`manifest.ps1`; then every machine
installs the exact new pin at its next sync (lockstep).
