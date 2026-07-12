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

### Commands

(準備中 — 設計書の「初期コンテンツ」参照)

### Skills

(準備中)

## 運用ルール

aidd と同じ: 1ファイル1関心事 / 2回ルール / README がインデックス / 重複禁止 / 受動ドキュメント禁止。詳細は[設計書](docs/superpowers/specs/2026-07-12-uidd-design.md)の「運用ルール」。
