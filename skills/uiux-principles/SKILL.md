---
name: uiux-principles
description: Use when making accessibility-related UI/UX design judgments that require verifiable WCAG criteria, such as contrast, target size, focus visibility, labels, or reflow. Read the Nielsen or Gestalt references only when a command explicitly requires them.
---

# UIUX 設計判断の契約と WCAG 要点

WCAG の数値・達成基準を伴う UI 設計判断で参照する。一般的な Nielsen ヒューリスティクスとゲシュタルト原則は、コマンドが明示した場合のみ `references/` から読む。全項目に出典 URL を付す (出典のない項目を追加しない)。

## 棲み分け (重要)

- **案件由来ルールがある場合はそれを優先**: 利用側プロジェクトから明示的に提供された確定判断は、このスキルより優先する (案件の確定判断 > 汎用原則)
- **機械検証可能な a11y チェックは `sb-verify`**: axe 等で検証できる項目の PASS/FAIL 判定はコマンド側の担当。このスキルは設計判断レベル (どう作るかの判断) のみを扱う
- **拡張方針**: 実案件で必要になった原則のみ追加する。包括カタログ化しない。追加時は出典 URL 必須、uidd 運用ルール3 に従い version を上げ CHANGELOG に記録する

## WCAG 実装判断の要点

出典: https://www.w3.org/WAI/WCAG22/quickref/

- **コントラスト (1.4.3 AA)**: 通常テキスト 4.5:1 以上、大きいテキスト (24px〜 / 太字 18.66px〜) 3:1 以上
- **色だけに頼らない (1.4.1)**: 状態・区別を色のみで伝えない。アイコン・テキスト・下線を併用する
- **ターゲットサイズ (2.5.8 AA)**: ポインタ操作のターゲットは最低 24×24 CSS px (間隔で補える場合を除く)
- **フォーカスの可視化 (2.4.7)**: キーボードフォーカスは常に視認可能にする。outline を消すなら代替表示必須
- **エラーの特定とラベル (3.3.1 / 3.3.2)**: エラー箇所をテキストで特定し、入力にはラベル・説明を付ける
- **リフロー (1.4.10)**: 320 CSS px 幅で横スクロールなしに利用できる (2次元コンテンツを除く)
