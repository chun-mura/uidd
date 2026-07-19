# uidd v0.6.0 — figma-implement + uiux-principles Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** uidd に「確定 Figma デザインからの忠実実装コマンド」と「出典照合済みの汎用 UIUX 原則スキル」を追加し v0.6.0 としてリリースする。

**Architecture:** マークダウン資産のみの変更。`commands/figma-implement.md` は既存 `commands/ui-propose.md` の構造 (frontmatter + 依存検出 + フロー + 成果物構成) を踏襲する。`skills/uiux-principles/SKILL.md` は `skills/ui-judgment/SKILL.md` と対になる汎用原則カタログで、reqd の requirements-notation と同じく全項目に出典 URL を付す。

**Tech Stack:** Claude Code プラグイン資産 (markdown)。検証は `jq` (manifest) と目視 + grep (必須節の存在確認)。テストスイートは無い (uidd に tests/ は存在しない)。

**Spec:** `docs/superpowers/specs/2026-07-19-figma-implement-uiux-principles-design.md`

## Global Constraints

- 会話・ドキュメント本文は日本語、コミットメッセージは英語 (グローバル CLAUDE.md)
- CI (`.github/workflows/validate.yml`): `commands/` または `skills/` の変更を push するとき、同一 push 内に `.claude-plugin/plugin.json` の version bump が必須 (運用ルール3)。**Task 3 完了まで push しないこと**
- 新規資産には README で未検証注記 ⚠️ を付ける (実案件で1回動作確認するまで)
- ui-judgment は案件由来ルールのみ・一般論禁止。uiux-principles との矛盾時は ui-judgment 優先 — この優先順位を両方の参照箇所で一貫させる
- コミット末尾に以下を付ける:
  ```
  Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_014pZoSQ5kFh19wTDLKsCZu3
  ```

---

### Task 1: `skills/uiux-principles/SKILL.md` — 汎用 UIUX 原則カタログ

**Files:**
- Create: `skills/uiux-principles/SKILL.md`

**Interfaces:**
- Produces: スキル名 `uiux-principles`。Task 2 の figma-implement と Task 3 の README がこの名前で参照する。参照順序の規約は「ui-judgment (案件由来・優先) → uiux-principles (汎用原則)」

- [ ] **Step 1: SKILL.md を作成する**

以下の内容で `skills/uiux-principles/SKILL.md` を作成する:

````markdown
---
name: uiux-principles
description: Use when making principle-based UI/UX decisions during implementation, proposal, or review work — information hierarchy, feedback, consistency, perceptual grouping, or accessibility-related design judgment. Provides a source-verified catalog of general UI/UX principles.
---

# 汎用 UIUX 原則カタログ

UI の実装・提案・レビューで原則ベースの判断が必要なとき、このカタログを参照する。全項目に出典 URL を付す (出典のない項目を追加しない)。

## 棲み分け (重要)

- **案件由来ルールは `ui-judgment`**: 実案件の実判断から育てるルール集。**このスキルと矛盾する場合は ui-judgment が優先** (案件の確定判断 > 汎用原則)
- **機械検証可能な a11y チェックは `sb-verify`**: axe 等で検証できる項目の PASS/FAIL 判定はコマンド側の担当。このスキルは設計判断レベル (どう作るかの判断) のみを扱う
- **拡張方針**: 実案件で必要になった原則のみ追加する。包括カタログ化しない。追加時は出典 URL 必須、uidd 運用ルール3 に従い version を上げ CHANGELOG に記録する

## Nielsen 10 ユーザビリティヒューリスティクス

出典: https://www.nngroup.com/articles/ten-usability-heuristics/

1. **システム状態の可視化**: 操作の結果・進行状況を適切な時間内にフィードバックする (ローディング・保存済み表示・選択状態)
2. **実世界との一致**: ユーザーに馴染みのある言葉・概念・順序を使う。内部用語を UI に漏らさない
3. **ユーザーの制御と自由**: やり直し・キャンセルの明確な出口を用意する (undo、閉じる、戻る)
4. **一貫性と標準**: 同じものは同じ見た目・同じ言葉にする。プラットフォーム慣習に従う
5. **エラーの防止**: エラー表示より発生自体の防止を優先する (確認、制約付き入力、危険操作の分離)
6. **再認 > 想起**: 選択肢・情報を見えるようにし、記憶に頼らせない
7. **柔軟性と効率**: 初心者向けの導線と熟練者向けのショートカットを両立させる
8. **美的で最小限のデザイン**: 判断に不要な情報を削る。情報の追加は既存情報との相対的競合
9. **エラーの認識・診断・回復支援**: エラーは平易な言葉で、原因と解決策を添えて示す
10. **ヘルプとドキュメント**: 必要なら文脈内で、タスクに即した短いヘルプを出す

## ゲシュタルト原則 (知覚グルーピング)

出典: https://www.nngroup.com/articles/gestalt-proximity/ / https://www.nngroup.com/articles/gestalt-similarity/

- **近接**: 近くに置かれた要素は同じグループと知覚される。関連要素の間隔 < 無関係要素の間隔、を必ず守る (余白の使い分けが枠線より優先)
- **類同**: 見た目 (色・形・サイズ) が同じ要素は同じ役割と知覚される。同じ機能は同じスタイル、違う機能に同じスタイルを使わない
- **共通領域**: 同じ背景・枠内の要素は1グループと知覚される。カード・セクション分割の根拠
- **連続・整列**: 整列された要素は関連づけて読まれる。グリッド・ベースラインを揃える

## WCAG 実装判断の要点

出典: https://www.w3.org/WAI/WCAG22/quickref/

- **コントラスト (1.4.3 AA)**: 通常テキスト 4.5:1 以上、大きいテキスト (24px〜 / 太字 18.66px〜) 3:1 以上
- **色だけに頼らない (1.4.1)**: 状態・区別を色のみで伝えない。アイコン・テキスト・下線を併用する
- **ターゲットサイズ (2.5.8 AA)**: ポインタ操作のターゲットは最低 24×24 CSS px (間隔で補える場合を除く)
- **フォーカスの可視化 (2.4.7)**: キーボードフォーカスは常に視認可能にする。outline を消すなら代替表示必須
- **エラーの特定とラベル (3.3.1 / 3.3.2)**: エラー箇所をテキストで特定し、入力にはラベル・説明を付ける
- **リフロー (1.4.10)**: 320 CSS px 幅で横スクロールなしに利用できる (2次元コンテンツを除く)
````

- [ ] **Step 2: 必須節の存在を検証する**

Run: `grep -c '^## ' skills/uiux-principles/SKILL.md && grep -c 'https://' skills/uiux-principles/SKILL.md`
Expected: 節数 `4` (棲み分け / Nielsen / ゲシュタルト / WCAG)、出典 URL を含む行数 `3` (grep -c は行数を数える。ゲシュタルト出典行は URL 2個で1行)

- [ ] **Step 3: コミット (push しない)**

```bash
git add skills/uiux-principles/SKILL.md
git commit -m "feat: add uiux-principles skill (source-verified general UI/UX catalog)"
```

---

### Task 2: `commands/figma-implement.md` — Figma → Storybook 忠実実装コマンド

**Files:**
- Create: `commands/figma-implement.md`

**Interfaces:**
- Consumes: Task 1 のスキル名 `uiux-principles` (参照順序: ui-judgment 優先 → uiux-principles)
- Produces: コマンド `/uidd:figma-implement`。Task 3 の README がファイル名 `figma-implement.md` で参照する

- [ ] **Step 1: figma-implement.md を作成する**

以下の内容で `commands/figma-implement.md` を作成する (構造は `commands/ui-propose.md` を踏襲):

````markdown
---
description: Figma のコンポーネントデザインから忠実なコンポーネント + ストーリーを対象プロジェクトに実装する
argument-hint: [Figma URL/ノード] [対象プロジェクトのパス]
---

対象: $ARGUMENTS

ui-propose が「要件から複数案を提案する」入口であるのに対し、このコマンドは「確定した Figma デザインから忠実に実装する」入口。対象はコンポーネント単位 (画面はコンポーネントの積み上げで対応する)。

## 依存検出 (処理前に必ず行う)

1. `$ARGUMENTS` が空、または Figma URL/ノードと対象プロジェクトのどちらかが欠けていれば AskUserQuestion で確認する。推測で続行しない
2. Figma MCP の有無を検出する: セッションに Figma 系 MCP ツール (get_code / get_variable_defs / get_image 等) が見えるかを確認する。無ければ AskUserQuestion でエクスポート画像のパスを確認し、「画像モード (忠実度低下) で続行する」と明示出力してから処理する。画像パスも得られなければ「デザインデータが取得できない」と明示して中断する
3. キャプチャ環境の有無を検出する: (a) セッションにブラウザ操作系 MCP ツール (Chrome MCP 等) が見えるか、(b) 対象プロジェクトの package.json に storybook 起動スクリプトがあるか。どちらか欠ければ「忠実度検証なしで続行する」と明示出力し、フロー5 をスキップして報告に「忠実度検証: 未実施 (環境なし)」と記載する

## フロー

1. **前提確認**: 資料・プロジェクトから以下が読み取れない場合、実装前に AskUserQuestion で確認する: フレームワーク (React / Vue 等)、既存デザイントークン・共通コンポーネントへの準拠要否、配置先ディレクトリ
2. **デザインデータ取得**: Figma MCP で対象ノードの構造・variables・Auto Layout・variant 構成を取得する。画像モードでは画像を読み込み、判別できない値 (正確な色コード・spacing 値等) は推測せず AskUserQuestion で確認する
3. **既存資産マッピング**: 取得した色・spacing・タイポグラフィ値を既存デザイントークンに、部品を既存共通コンポーネントに対応付ける。対応が無い値のみ新規化し、新規化した値は報告に列挙する (勝手にトークン体系を増やさない)
4. **実装**: 対象プロジェクトのフレームワークに合わせてコンポーネント + ストーリー (variant ごと) を実装する。UI 上の判断は skills/ui-judgment のルール (案件由来・優先) → skills/uiux-principles (汎用原則) の順で参照する。ルールにない新しい判断をしたら ui-judgment の昇格候補への記録を提案する (2回ルール)
5. **忠実度検証** (キャプチャ環境がある場合のみ): Storybook をローカル起動し各ストーリーをキャプチャして、Figma のエクスポート画像 (MCP の get_image または手元画像) と並べて比較する。保存先は対象プロジェクトのルートからの相対パス `docs/proposals/assets/YYYY-MM-DD-<slug>/<variant名>.png`。レイアウト・色・タイポグラフィの差分があれば修正し、再キャプチャしてから次へ進む。意図的にデザインから変えた点 (既存トークン準拠等) は差分として扱わず報告に記載する。起動したプロセスは検証後に停止する
6. **報告**: 実装ファイル・ストーリーのパス、キャプチャパス (未実施なら「忠実度検証: 未実施 (環境なし)」)、新規化したトークン・値の一覧、意図的にデザインから変えた点とその理由を報告する

**スコープ**: コンポーネント単位の実装まで。画面単位の一括分解・Figma への書き戻し (Code to Canvas)・Figma ファイル側の構造改善提案はスコープ外 (設計書参照)。
````

- [ ] **Step 2: 必須要素の存在を検証する**

Run: `grep -c 'AskUserQuestion' commands/figma-implement.md && grep -n 'ui-judgment\|uiux-principles' commands/figma-implement.md | head -5`
Expected: AskUserQuestion が `3` 箇所以上 (引数欠落・画像モード・不明値)、フロー4 に ui-judgment → uiux-principles の参照順序が存在

- [ ] **Step 3: コミット (push しない)**

```bash
git add commands/figma-implement.md
git commit -m "feat: add figma-implement command (faithful Figma-to-Storybook implementation)"
```

---

### Task 3: README インデックス・version 0.6.0・CHANGELOG

**Files:**
- Modify: `README.md` (commands 表・skills 表)
- Modify: `.claude-plugin/plugin.json` (version)
- Modify: `CHANGELOG.md` (先頭に 0.6.0 節)

**Interfaces:**
- Consumes: Task 1 `uiux-principles`、Task 2 `figma-implement.md`

- [ ] **Step 1: README の commands 表に行を追加する**

`README.md` の commands 表 (`sb-verify.md` の行の直後) に追加:

```markdown
| `figma-implement.md` | Figma URL/ノード + 対象プロジェクト → コンポーネント + ストーリー実装 | 確定 Figma デザインからの忠実実装: 依存検出 (Figma MCP / 画像モード) → デザインデータ取得 → 既存トークン・コンポーネントへのマッピング → 実装 → 忠実度検証 (キャプチャ環境がある場合) |
```

注: 表の直上にある未検証注記 ⚠️ (「実案件での動作確認前」) は表全体に係る既存文言のため変更不要。figma-implement もその対象に含まれる。

- [ ] **Step 2: README の skills 表に行を追加する**

`README.md` の skills 表 (`ui-judgment/` の行の直後) に追加:

```markdown
| `uiux-principles/` | UI の実装・提案・レビューで原則ベースの判断 (情報設計・フィードバック・一貫性・知覚グルーピング・a11y 設計判断) の前。出典照合済みの汎用原則カタログ。案件由来の ui-judgment と矛盾する場合は ui-judgment 優先 (⚠️ 実案件での動作確認前) |
```

- [ ] **Step 3: plugin.json の version を上げる**

`.claude-plugin/plugin.json` の `"version": "0.5.0"` を `"version": "0.6.0"` に変更する。

- [ ] **Step 4: CHANGELOG の先頭に 0.6.0 節を追加する**

`CHANGELOG.md` の `# Changelog` 直後に追加:

```markdown
## 0.6.0 - 2026-07-19

Per design spec 2026-07-19-figma-implement-uiux-principles-design.md:

- Add `commands/figma-implement.md` — faithful component implementation from a finalized Figma design (component granularity). Three-stage dependency detection following ui-propose: missing-argument AskUserQuestion, Figma MCP detection with image-mode fallback (explicit "画像モード (忠実度低下) で続行する" output; aborts if no image either), capture-environment detection gating the fidelity-verification step. Flow: design-data acquisition → mapping to existing tokens/components (new values are listed, never silently added) → implementation consulting ui-judgment (project rules, wins on conflict) then uiux-principles → capture-vs-Figma comparison with intentional deviations reported
- Add `skills/uiux-principles/` — source-verified catalog of general UI/UX principles (Nielsen 10 heuristics, Gestalt grouping, WCAG implementation points), every entry carries a source URL. Scoped as the general-knowledge counterpart to ui-judgment (project-derived, takes precedence on conflict); machine-verifiable a11y checks stay in sb-verify. Grows only from real-project need, no comprehensive cataloging
```

- [ ] **Step 5: manifest と整合性を検証する**

Run: `jq -e .version .claude-plugin/plugin.json && jq -e . .claude-plugin/marketplace.json > /dev/null && grep -c 'figma-implement\|uiux-principles' README.md`
Expected: `"0.6.0"`、marketplace.json は valid、README に両資産への参照が `2` 以上

- [ ] **Step 6: コミットして push する**

```bash
git add README.md .claude-plugin/plugin.json CHANGELOG.md
git commit -m "chore: release 0.6.0 (figma-implement command, uiux-principles skill)"
git push
```

push 後、CI (validate.yml) が「asset change is accompanied by a version bump」で成功することを確認する: `gh run watch --exit-status` または `gh run list --limit 1`。
