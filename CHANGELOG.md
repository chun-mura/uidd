# Changelog

## 0.3.0 - 2026-07-12

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
