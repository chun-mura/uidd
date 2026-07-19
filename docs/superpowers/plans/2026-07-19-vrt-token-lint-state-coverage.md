# uidd v0.7.0 — vrt / token-lint / sb-verify 状態網羅 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** VRT コマンド・トークン準拠検査コマンド・sb-verify の状態網羅チェック分離を実装し、優先度中・低の候補をマスター設計書に記録して v0.7.0 としてリリースする。

**Architecture:** マークダウン資産のみの変更。`commands/vrt.md` と `commands/token-lint.md` は既存コマンドの構造 (frontmatter + 依存検出 + フロー + 報告) を踏襲。`commands/sb-verify.md` はチェック1から状態網羅の責務を分離してチェック4を新設する編集。マスター設計書には「将来候補」節を追記する。

**Tech Stack:** Claude Code プラグイン資産 (markdown)。検証は `jq` (manifest) + grep (必須節)。テストスイートは無い。

**Spec:** `docs/superpowers/specs/2026-07-19-vrt-token-lint-state-coverage-design.md`

## Global Constraints

- ドキュメント本文は日本語、コミットメッセージは英語 (グローバル CLAUDE.md)
- CI (`.github/workflows/validate.yml`): `commands/` または `skills/` の変更を push するとき、同一 push 内に `.claude-plugin/plugin.json` の version bump が必須 (運用ルール3)。**Task 4 完了まで push しないこと**
- 新規コマンドは README の commands 表直上の既存 ⚠️ 注記 (実案件動作確認前) の対象
- 全依存検出は縮退・中断時に必ず明示出力する (無言のフォールバック禁止)
- vrt の baseline 更新は AskUserQuestion による承認必須 (承認なしの上書き禁止)
- 成果物パスは対象プロジェクトのルートからの相対パス、日付は `date +%F` (既存コマンドと同じ規約)
- コミット末尾に以下を付ける:
  ```
  Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_014pZoSQ5kFh19wTDLKsCZu3
  ```

---

### Task 1: マスター設計書に「将来候補 (未着手)」節を追記

**Files:**
- Modify: `docs/superpowers/specs/2026-07-12-uidd-design.md` (末尾に節を追加)

**Interfaces:**
- Produces: 将来候補の記録のみ。他タスクから参照されない (README には載せない)

- [ ] **Step 1: 設計書末尾に節を追加する**

`docs/superpowers/specs/2026-07-12-uidd-design.md` の末尾 (「スコープ外」節の後) に以下を追加する:

````markdown

## 将来候補 (未着手)

2026-07-19 のデザインプロセスギャップ分析より。いずれも実案件での実需要が2回発生したら昇格を検討する (2回ルール)。README には載せない (未実装資産をインデックスに混ぜない)。

### 優先度中

- **ヒューリスティック UI レビュー command**: uiux-principles + ui-judgment を評価軸に、ストーリー/キャプチャを対象とした原則ベースのレビュー成果物を出す。機械検証の sb-verify と対になる (aidd:design-review は汎用設計レビューのため棲み分け可能)
- **ハードコード値検出 hook**: 実装中に生値 (色 hex 等) を書いた時点で既存トークンを提案する受動的強制。運用ルール5 に整合。uidd 初の hook になる (aidd に前例あり)。token-lint (事後検査) の実行時版
- **提案成果物の共有 (プレビュー URL 発行・配布)**: フェーズ2 テンプレートリポジトリ側の責務 (本設計書「初期コンテンツ」節で定義済み)。テンプレートリポジトリ着手時に実装する

### 優先度低

- **figma-implement の画面単位対応**: 現在は意図的にコンポーネント単位のみ。積み上げ運用で不足が実際に出てから
- **Code to Canvas (実装 → Figma 書き戻し)**: デザイナーレビューを Figma 上で受けるフローには有効。需要が発生してから
- **UX ライティング/マイクロコピー検査**: まず uiux-principles の1節として薄く足すのが先。独立資産は2回ルール待ち
- **ユーザビリティテスト計画支援**: 上流は superpowers (brainstorming) / reqd と重なるため、置くなら「Storybook を被験材料にしたテスト計画」に限定する
````

- [ ] **Step 2: 追記を検証する**

Run: `grep -c '^### 優先度' docs/superpowers/specs/2026-07-12-uidd-design.md && grep -c '^- \*\*' docs/superpowers/specs/2026-07-12-uidd-design.md`
Expected: 優先度見出し `2`、候補項目の箇条書きを含む行が `7` 以上

- [ ] **Step 3: コミット (push しない)**

```bash
git add docs/superpowers/specs/2026-07-12-uidd-design.md
git commit -m "docs: record deferred feature candidates in master design spec"
```

---

### Task 2: `commands/vrt.md` — ビジュアルリグレッション検証コマンド

**Files:**
- Create: `commands/vrt.md`

**Interfaces:**
- Produces: コマンド `/uidd:vrt`。Task 4 の README がファイル名 `vrt.md` で参照する

- [ ] **Step 1: vrt.md を作成する**

以下の内容で `commands/vrt.md` を作成する:

````markdown
---
description: Storybook のビジュアルリグレッションを検証する (ツール検出 + ローカル基準のハイブリッド)
argument-hint: [対象ストーリーのパス。省略時は全件]
---

対象: $ARGUMENTS (未指定なら全ストーリー)

sb-verify が a11y・インタラクション・網羅を検証するのに対し、このコマンドは見た目の退行 (ビジュアルリグレッション) を検証する。

## 依存検出 (処理前に必ず行う)

1. Storybook 未導入 (`.storybook/` が無く package.json にも storybook が無い) なら「Storybook が導入されていない」と明示して中断する
2. **VRT ツール検出**: 対象プロジェクトの package.json に `chromatic` または `reg-suit` があるかを確認する。あればそのツールをオーケストレーションするモードで動作する (ローカル基準と二重管理しない)。両方あれば AskUserQuestion でどちらを使うか確認する
3. ツールが無ければ**ローカル基準モード**: キャプチャ環境を検出する: (a) セッションにブラウザ操作系 MCP ツール (Chrome MCP 等) が見えるか、(b) package.json に storybook 起動スクリプトがあるか。どちらか欠ければ、VRT はキャプチャなしで縮退できないため、欠けている依存とセットアップの選択肢 (ブラウザ MCP の導入 / Chromatic・reg-suit の導入) を明示して中断する

## ツールオーケストレーションモード (Chromatic / reg-suit 検出時)

1. ツールの実行コマンド (package.json の scripts、または `npx chromatic` / `npx reg-suit run`) を実行する。認証情報 (プロジェクトトークン等) が未設定で失敗した場合は、エラー内容とツールのセットアップ手順の参照先を明示して中断する。認証情報を推測・生成しない
2. ツールの出力 (変更検出件数・レビュー URL 等) を要約し、報告形式に従って報告する。差分の承認・却下はツール側の UI で行う (このコマンドは代行しない)

## ローカル基準モード

1. **baseline 確認**: 対象ストーリーごとに `docs/vrt/baselines/<story-id>.png` (対象プロジェクトのルートからの相対パス) の有無を確認する
2. **初回 baseline 作成**: baseline が無いストーリーは Storybook をローカル起動してキャプチャし、baseline として保存する。このストーリーは「baseline 作成 (比較なし)」として報告する
3. **比較**: baseline があるストーリーは現状をキャプチャする。現状キャプチャは `docs/vrt/current/<story-id>.png` に保存する。画像 diff ツール (ImageMagick `compare`・`pixelmatch` 等) が使えれば数値差分を取り、使えなければ「目視比較モード」と明示して両画像を並べて比較する
4. **判定**: 差分ありは FAIL とし、baseline と現状の両画像パスを記載する。差分なしは PASS
5. **baseline 更新**: FAIL 分は意図した変更か退行かをツールで判断できないため、**必ず AskUserQuestion で「意図した変更 (baseline 更新)」か「退行 (修正対象)」かの承認を得る**。承認なしで baseline を上書きしない
6. 起動したプロセスは検証後に停止する

## 報告形式

`docs/vrt/YYYY-MM-DD-vrt.md` (対象プロジェクトのルートからの相対パス、日付は `date +%F`) に保存し、パスを報告する:

```
## vrt 結果 (対象: <対象> / モード: <ツール名 or ローカル基準>)

### PASS (差分なし)
<ストーリー一覧>

### FAIL (差分あり)
<ストーリーごとに baseline / 現状の画像パスと差分の要約、ユーザー判断の結果 (更新 or 退行)>

### baseline 作成 (比較なし)
<初回キャプチャしたストーリー一覧>
```
````

- [ ] **Step 2: 必須要素の存在を検証する**

Run: `grep -c '明示して中断' commands/vrt.md && grep -c 'AskUserQuestion' commands/vrt.md`
Expected: 中断の明示が `3` 箇所 (Storybook 未導入 / キャプチャ環境欠落 / 認証未設定)、AskUserQuestion が `2` 箇所 (ツール両方検出時 / baseline 更新承認)

- [ ] **Step 3: コミット (push しない)**

```bash
git add commands/vrt.md
git commit -m "feat: add vrt command (tool-orchestration + local-baseline visual regression)"
```

---

### Task 3: `commands/token-lint.md` — トークン準拠検査コマンド

**Files:**
- Create: `commands/token-lint.md`
- Modify: `commands/ds-audit.md` (棲み分けポインタ1行追加)

**Interfaces:**
- Produces: コマンド `/uidd:token-lint`。Task 4 の README がファイル名 `token-lint.md` で参照する

- [ ] **Step 1: token-lint.md を作成する**

以下の内容で `commands/token-lint.md` を作成する:

````markdown
---
description: コードのデザイントークン準拠を検査し、Figma variables との乖離を報告する
argument-hint: [対象ディレクトリ]
---

対象: $ARGUMENTS

ds-audit がゼロからの棚卸し (重複コンポーネント含む広域・一回性) であるのに対し、このコマンドはトークン準拠の継続検査 (狭く・繰り返し使う)。figma-implement のトークンマッピング判断を、既存コード全体への検査として提供する。

## 依存検出 (処理前に必ず行う)

1. `$ARGUMENTS` が空なら AskUserQuestion で対象ディレクトリを確認する。パスが存在しなければ「対象ディレクトリが見つからない: <パス>」と明示して中断する。推測で続行しない
2. **トークン定義の検出**: 対象プロジェクトから CSS variables・Tailwind config・style-dictionary・theme ファイル等を探索する。見つからなければ AskUserQuestion でパスを確認し、それでも無ければ「トークン定義なし: ハードコード値の列挙のみ行う」と明示出力してから処理する (フロー1・3・4 をスキップ)
3. **Figma MCP 検出**: セッションに Figma 系 MCP ツールが見えるかを確認する。無ければ「Figma 差分なしで続行する」と明示出力し、フロー4 をスキップする

## フロー

1. **トークン定義読込**: 検出した定義から色・spacing・タイポグラフィ等のトークン一覧 (名前と値) を作る
2. **ハードコード値検出**: 対象コードから生値を検出する: 色 (hex・rgb/rgba・hsl)、spacing・サイズの px/rem 直値、font-size・font-weight 直値。トークン参照 (var()・theme 参照・トークン名の import) は対象外
3. **最近傍トークン提案**: 検出した各生値に最も近い既存トークンを提案する (色は色差、数値は近似値)。近いトークンが無い値は「トークン化候補」として件数の多い順に別掲する
4. **Figma 乖離 diff** (Figma MCP がある場合のみ): 対象の Figma ファイル/ノードを AskUserQuestion で確認して variables を取得し、コード側トークンと突き合わせる。値の乖離・片側にしか無い定義を一覧化する
5. **報告**: `docs/audit/YYYY-MM-DD-token-lint.md` (対象プロジェクトのルートからの相対パス、日付は `date +%F`) に保存し、パスを報告する

対象が大規模 (目安: 走査対象 50 ファイル超) な場合、ハードコード値の走査 (収集) はサブエージェントにファンアウトし、本体では集計と提案のみ行う。

## 成果物の構成

1. トークン定義の所在と件数 (定義なしの場合はその旨)
2. ハードコード値の検出一覧: ファイル:行 / 生値 / 提案トークン
3. トークン化候補 (近いトークンが無い値、件数順。トークン定義なしの場合は「トークン化候補: 未実施 (トークン定義なし)」と明記)
4. Figma 乖離 diff (MCP がある場合のみ。未実施なら「Figma 差分: 未実施 (MCP なし)」と明記)
````

- [ ] **Step 2: ds-audit.md に棲み分けポインタを追加する**

`commands/ds-audit.md` を読み、冒頭の説明部分 (frontmatter 直後の説明文の末尾) に以下の1文を追加する (既存の文章は変更しない):

```markdown
トークン準拠の継続検査 (ハードコード値検出・Figma 乖離) は `token-lint` の担当 (ds-audit は一回性の棚卸し)。
```

- [ ] **Step 3: 必須要素の存在を検証する**

Run: `grep -c 'AskUserQuestion' commands/token-lint.md && grep -c '明示' commands/token-lint.md && grep -c 'token-lint' commands/ds-audit.md`
Expected: AskUserQuestion `3` 箇所 (引数空 / トークン定義確認 / Figma 対象確認)、明示出力 `3` 箇所以上、ds-audit に token-lint 参照 `1` 以上

- [ ] **Step 4: コミット (push しない)**

```bash
git add commands/token-lint.md commands/ds-audit.md
git commit -m "feat: add token-lint command (token compliance check + Figma variables diff)"
```

---

### Task 4: sb-verify チェック4 分離・README・version 0.7.0・CHANGELOG

**Files:**
- Modify: `commands/sb-verify.md` (frontmatter description・チェック1・チェック4 新設・報告形式)
- Modify: `README.md` (commands 表: vrt / token-lint 追加、sb-verify 説明更新)
- Modify: `.claude-plugin/plugin.json` (version)
- Modify: `CHANGELOG.md` (先頭に 0.7.0 節)

**Interfaces:**
- Consumes: Task 2 `vrt.md`、Task 3 `token-lint.md`

- [ ] **Step 1: sb-verify.md のチェック1から状態網羅を分離しチェック4を新設する**

`commands/sb-verify.md` に以下の4編集を行う:

編集A — frontmatter の description を更新:

```
旧: description: Storybook のストーリー網羅・a11y・インタラクションテストを検証するオーケストレーター
新: description: Storybook のストーリー網羅・a11y・インタラクション・状態網羅を検証するオーケストレーター
```

編集B — オーケストレーション節の冒頭文を更新:

```
旧: 以下の3チェックを順に実行する。
新: 以下の4チェックを順に実行する。
```

編集C — チェック1を「ストーリー存在の網羅」に専念させる (状態の文をチェック4へ移す):

```
旧:
対象コンポーネントごとに検査する: ストーリーが存在するか。コンポーネントが持つ主要な状態 (variant 系 props、disabled、loading、error、empty 等、実装から読み取れるもの) がストーリー化されているか。
判定: 全コンポーネントにストーリーがあり主要状態がカバーされていれば PASS。不足があれば FAIL とし、不足しているコンポーネント・状態を列挙する。

新:
対象コンポーネントごとに検査する: ストーリーが存在するか (状態単位の網羅はチェック4 の担当)。
判定: 全コンポーネントにストーリーがあれば PASS。無いコンポーネントがあれば FAIL とし、列挙する。
```

編集D — チェック3の直後・「## 報告形式」の直前に、チェック4 の節を新設:

```markdown
### チェック4: 状態網羅

コンポーネントの props・条件分岐から表現可能な UI 状態 (variant 系 props、disabled、loading、error、empty 等、実装から読み取れるもの) を推定し、ストーリー未定義の状態を検出する。
判定: props・分岐に存在するのにストーリーが無い状態があれば FAIL とし、コンポーネントごとに不足状態を列挙する。全て揃っていれば PASS。hover・focus 等の疑似クラス状態は参考情報として列挙のみとし、FAIL の根拠にしない。
```

編集E — 報告形式のコードブロックにチェック4 の行を追加 (チェック3 の詳細行の直後):

```
### チェック4: 状態網羅 — PASS / FAIL
<詳細>
```

- [ ] **Step 2: README の commands 表を更新する**

編集A — `sb-verify.md` の行を更新:

```
旧: | `sb-verify.md` | 対象ストーリー (省略時全件) → チェック別 PASS/FAIL ブロック | Storybook 検証のオーケストレーター: ストーリー網羅 / a11y / インタラクションの3チェックを独立実行・独立報告 |
新: | `sb-verify.md` | 対象ストーリー (省略時全件) → チェック別 PASS/FAIL ブロック | Storybook 検証のオーケストレーター: ストーリー網羅 / a11y / インタラクション / 状態網羅の4チェックを独立実行・独立報告 |
```

編集B — `figma-implement.md` の行の直後に2行追加:

```markdown
| `vrt.md` | 対象ストーリー (省略時全件) → `docs/vrt/<日付>-vrt.md` | ビジュアルリグレッション検証: Chromatic / reg-suit を検出すればオーケストレーション、無ければローカル基準モード (baseline 保存 → キャプチャ比較。baseline 更新はユーザー承認必須)。キャプチャ環境が無ければ中断 |
| `token-lint.md` | 対象ディレクトリ → `docs/audit/<日付>-token-lint.md` | トークン準拠の継続検査: ハードコード値検出 → 最近傍トークン提案 → Figma MCP があれば variables 乖離 diff (ds-audit は一回性の棚卸し、こちらは繰り返し使う) |
```

- [ ] **Step 3: plugin.json の version を上げる**

`.claude-plugin/plugin.json` の `"version": "0.6.1"` を `"version": "0.7.0"` に変更する。

- [ ] **Step 4: CHANGELOG の先頭に 0.7.0 節を追加する**

`CHANGELOG.md` の `# Changelog` 直後に追加する:

```markdown
## 0.7.0 - 2026-07-19

Per design spec 2026-07-19-vrt-token-lint-state-coverage-design.md (design-process gap analysis, high-priority items):

- Add `commands/vrt.md` — visual regression verification. Detects Chromatic / reg-suit and orchestrates the tool when present (no double bookkeeping); otherwise runs local-baseline mode (baseline save → capture → diff, numeric when a diff tool exists, explicit visual-comparison mode otherwise). Baseline updates always require user approval via AskUserQuestion; aborts with setup guidance when no capture environment exists (the one command that cannot degrade without captures)
- Add `commands/token-lint.md` — continuous token-compliance check: hardcoded-value detection (colors, spacing, typography) → nearest-token proposal → Figma variables drift diff when Figma MCP is present. Scoped against ds-audit (one-shot broad inventory) with mutual pointers in both files
- `sb-verify`: split state coverage out of check 1 into a new check 4 (one concern per check — check 1 already inspected major states, so a plain addition would have duplicated it). Check 1 now verifies story existence only; check 4 fails on states present in props/branches but missing stories, and lists pseudo-class states (hover/focus) as informational only
- Master design spec: record deferred candidates (heuristic UI review command, hardcoded-value hook, proposal sharing, screen-level figma-implement, Code to Canvas, UX writing, usability-test planning) with promotion criteria (2回ルール)
```

- [ ] **Step 5: manifest と整合性を検証する**

Run: `jq -e .version .claude-plugin/plugin.json && jq -e . .claude-plugin/marketplace.json > /dev/null && grep -c 'vrt.md\|token-lint.md' README.md && grep -c '4チェック' README.md`
Expected: `"0.7.0"`、marketplace.json は valid、README に新コマンド参照 `2` 以上、「4チェック」 `1`

- [ ] **Step 6: コミットして push する**

```bash
git add commands/sb-verify.md README.md .claude-plugin/plugin.json CHANGELOG.md
git commit -m "chore: release 0.7.0 (vrt, token-lint, sb-verify state-coverage split)"
git push
```

push 後、CI (validate.yml) の成功を確認する: `gh run watch --exit-status` または `gh run list --limit 1`。
