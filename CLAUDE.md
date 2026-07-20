# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

uidd is a Claude Code **plugin** (not an app) distributed via `/plugin marketplace add chun-mura/uidd`. It ships `commands/` (task-specific prompts, invoked as `/uidd:*`) and `skills/` (auto-triggered judgment references). There is no build, no package.json, no test suite — the "code" is markdown prompt files, and the repo's own CI validates *those*, not application logic. Full rationale lives in `docs/superpowers/specs/2026-07-12-uidd-design.md`; changelog history (which explains *why* each shape decision was made) is in `CHANGELOG.md`.

## Where uidd sits relative to aidd / superpowers

Three-layer split (see design spec "aidd / superpowers との棲み分け"):

- **superpowers** — how to work (process: TDD, debugging, planning)
- **aidd** — generic dev tasks (design-review, test-perspectives, ADR, reviewer)
- **uidd** — UI-domain-specific tasks (this repo: judgment rules, proposal flow, Storybook verification)

uidd commands/skills must run standalone even if aidd/superpowers are not installed — **never call aidd at runtime**, only reference it via doc pointers. If a future asset needs to call aidd at runtime, it must detect aidd's absence and abort with an explicit error (same degrade contract as MCP detection below). Do not duplicate anything superpowers or aidd already own — check there before adding a new command/skill here.

## Operating rules (shared with aidd — do not relitigate these per-PR)

1. One file, one concern. Every doc opens with "when to use this."
2. **2回ルール (rule of two)**: nothing gets promoted into `commands/` or `skills/` until it has been used twice on a real project. `ui-judgment` is frozen while it has no real rules; reactivate it only when repeated, evidence-backed project judgments can justify it. Do not add general/unsourced UI opinions as project rules.
3. **README is the index.** Any time a command/skill is added, changed, or removed: update `README.md`'s table AND bump `version` in `.claude-plugin/plugin.json` AND add a `CHANGELOG.md` entry. CI (`.github/workflows/validate.yml`) enforces the version bump — a PR touching `commands/` or `skills/` without a `"version"` diff in `plugin.json` fails.
4. No duplication with superpowers/aidd — pointer only.
5. No passive documentation — anything meant to influence a session must be a skill (or hook), not a doc nobody reads.
6. When `ui-judgment` is reactivated, every entry must cite the real project decision that justified it (`根拠: <案件・判断の要約>, <date>`). Do not require writes to an installed plugin cache during a client-project session; collect evidence through the project workflow and update the plugin source only when reactivation is justified.

## Verifying changes

There's no app to run. Verification = matches these commands and skills:

```bash
jq -e . .claude-plugin/plugin.json          # manifest must be valid JSON
jq -e . .claude-plugin/marketplace.json     # marketplace entry must be valid JSON
```

This is exactly what `.github/workflows/validate.yml` runs on push/PR to `main`, plus the version-bump check described in rule 3 above (diffs `commands/`/`skills/` against the base ref and requires a `"version"` line change in `plugin.json`'s diff). Run both `jq` checks locally before finishing any change that touches `.claude-plugin/*.json`.

There is no linter/formatter for the markdown prompt files themselves — review them by reading, matching the existing structure (frontmatter → 対象 → 依存検出 → フロー/オーケストレーション → 出力/報告形式).

## Command architecture (read before editing any `commands/*.md`)

Every command follows the same shape: YAML frontmatter (`description`, `argument-hint`) → `対象: $ARGUMENTS` → a **依存検出 (dependency detection)** section that runs before any real work → the main flow → an output/report section. The dependency-detection step is not optional boilerplate — it is the degrade contract: each command detects its dependencies (Storybook MCP, Figma MCP, capture/browser MCP, VRT tooling, token definitions) up front and either explicitly announces a degraded mode ("ファイル直読みモードで続行する", "画像モード (忠実度低下) で続行する", "キャプチャなしで続行する") or aborts with a clear message. Never silently continue past a missing dependency, and never invent/guess a value a human should confirm (paths, colors, tokens) — use `AskUserQuestion`.

Command relationships (who feeds whom, and why they're not merged):

- `ui-propose` (要件 → 複数案) and `figma-implement` (確定 Figma デザイン → 忠実実装) are the two implementation entry points — one explores options, the other implements a fixed design 1:1. Both write generated code + Storybook stories. Explicitly provided project rules take precedence; `figma-implement` reads `uiux-principles` and its specific references only when a general principle is needed.
- `ds-audit` (one-shot, broad: colors + spacing + duplicate components → `docs/audit/<date>-audit.md`) vs `token-lint` (repeated, narrow: hardcoded-value → nearest-token + Figma variables drift → `docs/audit/<date>-token-lint.md`). Don't extend one to cover the other's job — they're intentionally scoped apart and cross-reference each other in their own files.
- `sb-verify` is an orchestrator over 4 independent checks (story coverage, a11y, interaction, state coverage) — each reports its own PASS/FAIL block; one check failing or being unrunnable must never block the others. `vrt` is visual-regression only, deliberately separate from `sb-verify` (different concern: pixel diff vs behavior/coverage), and either orchestrates a detected tool (Chromatic/reg-suit — never double-books against local baselines) or falls back to a local-baseline mode where **baseline overwrites always require `AskUserQuestion` approval**, and the run aborts entirely if no capture environment exists (the one command that cannot degrade further).
- Large targets (rough thresholds noted per-command: >10 files/components, >50 files for token-lint) fan collection out to subagents; aggregation/judgment always stays in the main context — don't move judgment into the fan-out.

## UI judgment precedence

`ui-judgment` is frozen until real-project evidence justifies reactivation. When a client project explicitly provides a confirmed UI rule, it wins over `uiux-principles`. `uiux-principles` retains WCAG design criteria; Nielsen heuristics and Gestalt grouping are on-demand references, not auto-triggered catalog content. Machine-verifiable a11y checks (axe, alt text, label association) belong in `sb-verify`, not in `uiux-principles`.

## Commands are unverified — this matters when editing them

README's commands table carries the note: **⚠️ commands are pre-real-project-verification**. Per the design spec's error-handling section, each command needs at least one real-project run before that warning is removed from README. If you're asked to "fix" a command, check whether the issue is a real bug or just an unverified assumption — and don't remove the warning yourself; that only happens after an actual verified run, and should be called out explicitly to the user rather than silently dropped in a broader edit.

## README operating rule parity with aidd

README explicitly states its operating rules are "aidd と同じ" (same as aidd): one-file-one-concern, 2回ルール, README-as-index, no-duplication, no-passive-docs. If aidd's own operating rules evolve, check whether uidd's README/this file need the same update — they're meant to stay in lockstep, not drift independently.
