---
name: ui-judgment
description: Use when making UI design decisions during implementation or proposal work — spacing, visual hierarchy, component state design, or component/props naming. Applies accumulated project-derived judgment rules before deciding.
---

# UI 判断ルール集

UI の実装・提案で判断 (余白・階層・状態設計・命名) を行う前に、このルール集を参照する。該当ルールがあればそれに従う。該当ルールがなく新しい判断をした場合は、まず「昇格候補」への記録をユーザーに提案する (2回ルール)。

## 運用 (重要)

- ルールは**実案件での実判断のみ**を根拠とする。根拠のない一般論を書かない
- **2回ルールのカウント**: 新しい判断は 1回目は「昇格候補」に記録する。候補と同種の判断が 2回目に出たら、該当カテゴリの本ルールへの昇格をユーザーに提案し、候補から削除する
- 案件セッション中に得た候補・ルールは必ず uidd 本体 (このファイル) に還元してから各案件へ反映する。uidd 本体が単一の真実源であり、案件側ローカルの独自追記を正としない
- ルール形式: `- <ルール本文> (根拠: <案件・判断の要約>, <YYYY-MM-DD>)`
- 候補形式: `- <判断内容> (初出: <案件・文脈の要約>, <YYYY-MM-DD>)`
- 追記時は uidd の運用ルール3 に従い plugin.json の version を上げ、CHANGELOG に記録する

## 余白

(実案件の判断が入るまで空。一般論を書かない)

## 階層

(実案件の判断が入るまで空。一般論を書かない)

## 状態設計

(実案件の判断が入るまで空。一般論を書かない)

## 命名

(実案件の判断が入るまで空。一般論を書かない)

## 昇格候補 (1回目の判断の記録)

まだ本ルールに昇格していない判断。同種の判断が 2回目に出たら上のカテゴリへ昇格させる。

(候補が入るまで空)
