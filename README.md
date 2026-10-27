# uidd — UI-Driven Development Toolkit

Storybook を中心とした UI 提案・デザインシステム構築ワークフローのプラグイン (Claude Code・Codex・Cursor で使える)。デザインシステム / UI パーツ / Storybook の作成・更新・レビュー・検証と、クライアント向け提案フローを仕組み化する。

**棲み分け**: process は superpowers、汎用 task (design-review / test-perspectives 等) は [aidd](https://github.com/chun-mura/aidd)、UI ドメイン特化が uidd。詳細は [設計書](docs/superpowers/specs/2026-07-12-uidd-design.md)。

## 導入

### 個人で使う

```bash
# Claude Code
/plugin marketplace add chun-mura/uidd
/plugin install uidd@uidd

# Codex CLI
codex plugin marketplace add chun-mura/uidd
codex plugin add uidd@uidd

# Cursor agent CLI (ローカルの clone を読み込む)
cursor-agent --plugin-dir <uidd を clone したパス>

# 併用推奨: aidd (汎用タスク) + superpowers (実装プロセス)
```

Cursor は、Claude Code に入れたプラグインの skill も読み込む。Claude Code に uidd を入れてあれば、Cursor 側で別に入れなくても使える。

呼び出し方は、Claude Code が `/uidd:<名前>`、Codex が `$uidd:<名前>`、Cursor が `/<名前>` (例: `/uidd:token-lint src/`・`$uidd:token-lint src/`・`/token-lint src/`)。

ユーザーへの確認 (`AskUserQuestion`) を前提にした手順は、質問ツールが無いハーネスでは「質問を書いて止まり、回答を待つ」として動く。Codex の質問ツールは既定で Plan mode でしか使えないため、Codex では回答のたびに会話を続ける形になる。

### クライアント向け公開

uidd リポジトリは public のため、クライアントは自身の権限追加なしに導入できる。

1. **配布形式**: クライアントに `chun-mura/uidd` を marketplace として案内する。リポジトリへの書き込み権限は不要 (public リポジトリの clone/参照のみ)
2. **クライアント側の導入手順**: 上記「個人で使う」と同じ2コマンド (`/plugin marketplace add chun-mura/uidd` → `/plugin install uidd@uidd`) を案内する
3. **ライセンス**: [MIT](LICENSE)。再配布・改変を制限しない
4. **更新の配信**: `plugin.json` の `version` に固定配信される。リリース時は必ず version を上げて CHANGELOG に記録する (運用ルール3)。クライアント側は `/plugin update uidd` で追従する。破壊的変更を出す場合は CHANGELOG に明記し、事前に連絡する
5. **プロジェクト単位の自動導入**: クライアント案件のリポジトリに書き込み権限があり、チーム全員へ自動でインストール提案したい場合は、aidd 運用ルール8 と同じ方式 (`.claude/settings.json` への team-settings マージ) を検討する。uidd 側の team-settings テンプレートは未整備 (必要になったタイミングで追加する)

## インデックス

### Skills (利用者が呼ぶ。モデルからは自動起動しない)

> ⚠️ 以下の skills は実案件での動作確認前 (未検証)。設計書エラーハンドリング節に従い、実案件で1回動作確認したらこの注記を外す。0.8.0 まではコマンド (`commands/*.md`) だった。

| ファイル | 入力 → 出力 | 説明 |
|---------|-----------|------|
| `ui-propose/` | 対象画面/要件資料のパス → `docs/proposals/<日付>-<slug>.md` | 提案フロー: 前提確認 → 資料読込 → 調査 → 複数案実装 → レンダリング検証 (キャプチャ環境がある場合) → 理由・仕様・キャプチャ付き提案成果物の生成 (共有はフェーズ2) |
| `ds-audit/` | 対象ディレクトリ → `docs/audit/<日付>-audit.md` | 既存フロントエンドの棚卸し: 色・spacing・重複コンポーネントを抽出し統合候補を提案 |
| `sb-verify/` | 対象ストーリー (省略時全件) → チェック別 PASS/FAIL ブロック | Storybook 検証のオーケストレーター: ストーリー網羅 / a11y / インタラクション / 状態網羅の4チェックを独立実行・独立報告 |
| `figma-implement/` | Figma URL/ノード + 対象プロジェクト → コンポーネント + ストーリー実装 | 確定 Figma デザインからの忠実実装: 依存検出 (Figma MCP / 画像モード) → デザインデータ取得 → 既存トークン・コンポーネントへのマッピング → 実装 → 忠実度検証 (キャプチャ環境がある場合) |
| `vrt/` | 対象ストーリー (省略時全件) → `docs/vrt/<日付>-vrt.md` | ビジュアルリグレッション検証: Chromatic / reg-suit を検出すればオーケストレーション、無ければローカル基準モード (baseline 保存 → キャプチャ比較。baseline 更新はユーザー承認必須)。キャプチャ環境が無ければ中断 |
| `token-lint/` | 対象ディレクトリ → `docs/audit/<日付>-token-lint.md` | トークン準拠の継続検査: ハードコード値検出 → 最近傍トークン提案 → Figma MCP があれば variables 乖離 diff (ds-audit は一回性の棚卸し、こちらは繰り返し使う) |

### Skills (自動トリガする知見)

| ファイル | 発火条件 |
|---------|------|
| `uiux-principles/` | WCAG の数値・達成基準を伴う a11y 設計判断の前。Nielsen ヒューリスティクスとゲシュタルト原則は、コマンドが必要時にのみ references から明示読込する (⚠️ 実案件での動作確認前) |

## 運用ルール

aidd と同じ: 1ファイル1関心事 / 2回ルール / README がインデックス / 重複禁止 / 受動ドキュメント禁止。詳細は[設計書](docs/superpowers/specs/2026-07-12-uidd-design.md)の「運用ルール」。
