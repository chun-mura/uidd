# uidd — UI-Driven Development Toolkit

Storybook を中心とした UI 提案・デザインシステム構築ワークフローの Claude Code プラグイン。デザインシステム / UI パーツ / Storybook の作成・更新・レビュー・検証と、クライアント向け提案フローを仕組み化する。

**棲み分け**: process は superpowers、汎用 task (design-review / test-perspectives 等) は [aidd](https://github.com/chun-mura/aidd)、UI ドメイン特化が uidd。詳細は [設計書](docs/superpowers/specs/2026-07-12-uidd-design.md)。

## 導入

```bash
/plugin marketplace add chun-mura/uidd
/plugin install uidd@uidd

# 併用推奨: aidd (汎用タスク) + superpowers (実装プロセス)
```

## インデックス

### Commands (タスク特化プロンプト)

> ⚠️ 以下の commands は実案件での動作確認前 (未検証)。設計書エラーハンドリング節に従い、実案件で1回動作確認したらこの注記を外す。

| ファイル | 入力 → 出力 | 説明 |
|---------|-----------|------|
| `ui-propose.md` | 対象画面/要件資料のパス → `docs/proposals/<日付>-<slug>.md` | 提案フロー: 前提確認 → 資料読込 → 調査 → 複数案実装 → 理由・仕様付き提案成果物の生成 (共有はフェーズ2) |
| `ds-audit.md` | 対象ディレクトリ → `docs/audit/<日付>-audit.md` | 既存フロントエンドの棚卸し: 色・spacing・重複コンポーネントを抽出し統合候補を提案 |
| `sb-verify.md` | 対象ストーリー (省略時全件) → チェック別 PASS/FAIL ブロック | Storybook 検証のオーケストレーター: ストーリー網羅 / a11y / インタラクションの3チェックを独立実行・独立報告 |

### Skills (自動トリガする知見)

| ファイル | 発火条件 |
|---------|------|
| `ui-judgment/` | UI 判断 (余白・階層・状態設計・命名) の前。実案件の判断を追記して育てる (初期は空) |

## 運用ルール

aidd と同じ: 1ファイル1関心事 / 2回ルール / README がインデックス / 重複禁止 / 受動ドキュメント禁止。詳細は[設計書](docs/superpowers/specs/2026-07-12-uidd-design.md)の「運用ルール」。
