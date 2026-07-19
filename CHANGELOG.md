# Changelog

## 0.6.0 - 2026-07-19

Per design spec 2026-07-19-figma-implement-uiux-principles-design.md:

- Add `commands/figma-implement.md` — faithful component implementation from a finalized Figma design (component granularity). Three-stage dependency detection following ui-propose: missing-argument AskUserQuestion, Figma MCP detection with image-mode fallback (explicit "画像モード (忠実度低下) で続行する" output; aborts if no image either), capture-environment detection gating the fidelity-verification step. Flow: design-data acquisition → mapping to existing tokens/components (new values are listed, never silently added) → implementation consulting ui-judgment (project rules, wins on conflict) then uiux-principles → capture-vs-Figma comparison with intentional deviations reported
- Add `skills/uiux-principles/` — source-verified catalog of general UI/UX principles (Nielsen 10 heuristics, Gestalt grouping, WCAG implementation points), every entry carries a source URL. Scoped as the general-knowledge counterpart to ui-judgment (project-derived, takes precedence on conflict); machine-verifiable a11y checks stay in sb-verify. Grows only from real-project need, no comprehensive cataloging

## 0.5.0 - 2026-07-17

From external plugin review (adopted findings 1 and 3):

- `ui-propose`: add a rendering-verification step — when a browser MCP and a local Storybook script are both detected, each proposal story is rendered, screenshotted to `docs/proposals/assets/`, fixed on visual defects, and the captures are embedded in the proposal artifact; without a capture environment the command states "キャプチャなしで続行する" and the artifact records "キャプチャ: 未実施 (環境なし)" (same degrade contract as the other detections)
- `ui-judgment`: add a 昇格候補 section implementing the 2回ルール count — a first-time judgment is recorded as a candidate; on its second occurrence it is promoted to a real rule and removed from candidates. `ui-propose` now proposes candidate recording instead of direct rule addition

## 0.4.0 - 2026-07-12

Command refinements and CI, from plugin self-review:

- `sb-verify`: check 1 now uses Storybook MCP for component/story enumeration when detected (the detection result was previously unused); static a11y mode counts its findings as violations, and excludes contrast (not statically verifiable — reported as 未検証); large targets fan out collection to subagents
- `ds-audit`: large targets fan out collection to subagents; aggregation stays in the main context
- `ui-propose` / `ds-audit`: empty-argument runs now ask for the target via AskUserQuestion instead of falling into the missing-path error; output paths clarified as relative to the target project root
- Add `.github/workflows/validate.yml`: manifest JSON validity + version-bump check when commands/skills change (operating rule 3)
- Design spec: status updated to "phase 1 implemented"; `docs/tips/` marked as phase-2 (not yet created)

Client-facing distribution readiness (per design spec スコープ外 note: "公開時に README へ追記"):

- Add MIT `LICENSE`
- `plugin.json` gains `homepage`/`repository`/`license`/`keywords`; marketplace entry gains `category`/`tags`
- README gains a "クライアント向け公開" section: distribution steps, license, version-pinned update flow, and a pointer to aidd's team-settings pattern for project-wide auto-install

## 0.2.0 - 2026-07-12

Phase 1 initial assets (per design spec 2026-07-12-uidd-design.md):

- Add `commands/ui-propose.md` — multi-option UI proposal flow (artifact generation only)
- Add `commands/ds-audit.md` — color / spacing / duplicate-component audit of existing frontends
- Add `commands/sb-verify.md` — Storybook verification orchestrator (3 independent checks)
- Add `skills/ui-judgment/` — UI judgment rule collection (empty scaffold; rules require real project decisions)
- Commands are listed in the README with a "not yet verified in a real project" note until first real use

## 0.1.0 - 2026-07-12

- Initial scaffold: plugin manifest, marketplace, README, design spec
