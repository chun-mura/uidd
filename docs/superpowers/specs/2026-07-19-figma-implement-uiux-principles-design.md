# uidd v0.6.0 — figma-implement コマンド + uiux-principles スキル 設計

日付: 2026-07-19
ステータス: 設計承認済み・実装前

## 目的

uidd に UI 実装の新しい入口と判断知見を追加する:

1. **figma-implement コマンド**: 確定した Figma デザインから、なるべく忠実なコンポーネント + ストーリーを対象プロジェクトに実装する。既存の `ui-propose` (要件から複数案を提案する入口) と対になる「確定デザインから実装する入口」
2. **uiux-principles スキル**: 出典照合済みの汎用 UIUX 原則カタログ。`ui-judgment` (案件由来ルール) が設計上除外している一般知識を、別の真実源として提供する

## 背景 (価値の根拠)

- Figma 公式 MCP サーバーは Claude Code との双方向連携に対応し、variables / Auto Layout / variant 構造 / Code Connect を構造化データとして取得できる (2026-02 時点)。画像からの推測に比べ忠実な再現が実用水準
  - 出典: https://help.figma.com/hc/en-us/articles/39888612464151 / https://www.figma.com/blog/introducing-claude-code-to-figma/
- 忠実度はデザインファイル側の構造 (Auto Layout・variant 命名・semantic token) に強く依存する。出典: https://blog.logrocket.com/ux-design/design-to-code-with-figma-mcp/
- Figma MCP の利用に Figma 側のプラン・シート条件がある可能性は**未確認**。実案件導入時に確認する

## 既存資産との棲み分け

| 資産 | 役割 | 新規資産との関係 |
|---|---|---|
| `ui-propose` | 要件 → 複数案提案 | figma-implement は確定デザイン → 忠実実装。依存検出パターンを共有する |
| `sb-verify` | 機械検証可能な a11y・インタラクション検証 | figma-implement の成果物を検証する出口。uiux-principles は設計判断レベルの原則適用のみ担当し重複しない |
| `ui-judgment` | 案件由来の判断ルール (一般論禁止・2回ルール) | uiux-principles は汎用原則の真実源。**矛盾時は ui-judgment が優先** |

## コンポーネント1: `commands/figma-implement.md`

入力: Figma URL/ノード (コンポーネント単位) + 対象プロジェクト → 出力: コンポーネント + ストーリー実装

### 依存検出 (ui-propose と同型の3段検出)

1. 引数が空なら AskUserQuestion で Figma URL/ノードと対象プロジェクトを確認する。推測で続行しない
2. Figma MCP の有無を検出する。無ければエクスポート画像のパスを確認し「画像モード (忠実度低下) で続行する」と明示出力してから処理する
3. キャプチャ環境の有無を検出する (ui-propose と同じ判定: ブラウザ操作系 MCP + storybook 起動スクリプト)。欠ければ忠実度検証をスキップし、成果物に「忠実度検証: 未実施 (環境なし)」と記載する

### フロー

1. **前提確認**: フレームワーク、既存デザイントークン・共通コンポーネント準拠か、配置先。資料から読み取れなければ AskUserQuestion
2. **デザインデータ取得**: MCP で variables / Auto Layout / variant 構造を取得する。画像モードでは画像を読み込み、判別できない値 (正確な色・spacing 等) は AskUserQuestion で確認する
3. **既存資産マッピング**: Figma の値を既存トークン・共通コンポーネントに対応付ける。対応が無い値のみ新規化し、その旨を成果物に記録する (勝手にトークン体系を増やさない)
4. **実装**: コンポーネント + ストーリー。UI 判断は ui-judgment (案件由来・優先) → uiux-principles (汎用原則) の順で参照する。新しい判断は ui-judgment の昇格候補への記録を提案する (2回ルール)
5. **忠実度検証** (キャプチャ環境がある場合のみ): Storybook をローカル起動しキャプチャを取得、Figma エクスポート画像と比較する。差分があれば修正 → 再キャプチャ。保存先・プロセス停止は ui-propose のレンダリング検証と同じ規則
6. **報告**: 実装ファイル・ストーリー・キャプチャパス + 忠実度差分の注記 (意図的に変えた点を明記)

### スコープ外

- 画面単位の一括分解 (コンポーネントの積み上げで対応)
- Figma への書き戻し (Code to Canvas)
- Figma ファイル側の構造改善提案

## コンポーネント2: `skills/uiux-principles/SKILL.md`

- **発火条件**: UI の実装・提案・レビューで原則ベースの判断 (情報設計・フィードバック・一貫性・知覚グルーピング等) が必要なとき
- **内容 (厳選コアのみ)**: Nielsen 10 ヒューリスティクス / ゲシュタルト主要原則 / WCAG の実装判断に効く要点。**全項目に出典 URL を付す** (reqd の requirements-notation と同型)。拡張は実案件で必要になった原則のみ追加する (包括カタログ化しない)
- **棲み分けをスキル冒頭に明記**:
  - 案件由来ルール = ui-judgment (矛盾時はそちらが優先)
  - 機械検証可能な a11y チェック = sb-verify (このスキルは設計判断レベルのみ)

## 付随変更

- README インデックス更新: commands 表に figma-implement、skills 表に uiux-principles を追加。新規分に未検証注記 ⚠️ を付与 (既存慣行)
- `plugin.json` version 0.6.0、CHANGELOG 追記

## エラーハンドリング

- 両資産とも実案件での動作確認前は未検証注記を付け、1回動作確認したら外す (既存運用と同じ)
- figma-implement は各依存検出の縮退時に必ず明示出力する (無言のフォールバック禁止)
