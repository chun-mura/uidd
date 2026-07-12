# テスト観点: v0.4.0 コマンド磨き込み + CI 追加

日付: 2026-07-12
対象: commands 3ファイルの磨き込み、`.github/workflows/validate.yml` 新規、設計書 status 更新、plugin.json 0.4.0 bump、CHANGELOG

実行可能コードは CI ワークフローのみ。Markdown プロンプト資産は単体テスト不能のため、観点は「実案件初回検証時の確認項目」として列挙する (uidd フェーズ1 宿題1 と同時に消化する)。

## validate.yml (CI)

### 正常系

- **must**: 資産変更 (commands/ or skills/) + plugin.json version 変更が同一範囲にある push → bump チェック PASS — **カバー済み** (ローカルで作業ツリーに対しロジックをシミュレート済み)
- **must**: 資産変更を含まない push (docs のみ等) → bump チェックはスキップ扱いで PASS
- **must**: マニフェスト2ファイルが valid JSON → validate ステップ PASS — **カバー済み** (`jq -e` 実行済み)

### 境界値

- **should**: 初回 push / force push で `event.before` が all-zero または未知 SHA → チェックをスキップして exit 0 (ジョブを落とさない)
- **should**: skills/ のみの変更でも bump が要求される (正規表現 `^(commands/|skills/)` の skills 側)
- **could**: 資産変更 + plugin.json の description のみ変更 (version 行は非変更) → FAIL する (`grep '"version"'` が version 行の diff を要求)

### 異常系

- **must**: 資産変更あり・version bump なし → exit 1 でジョブが FAIL する
- **must**: plugin.json が壊れた JSON → `jq -e` が非ゼロで FAIL する

### 状態遷移

- **could**: fork からの pull_request でも `origin/<base_ref>` が参照可能 (fetch-depth: 0 で全履歴取得のため成立する想定)

### 実施方法

CI はローカル実行不能のため、must 観点は次回 push (本コミット) の Actions 実行結果で正常系を、検証用ブランチ (資産のみ変更して push → FAIL 確認) で異常系を確認する。

## Markdown プロンプト資産 (実案件初回検証時の確認項目)

- **must**: `ui-propose` / `ds-audit` を引数なしで実行 → 「対象が見つからない: (空)」ではなく AskUserQuestion で対象を確認する
- **should**: `sb-verify` 静的検査モード → 報告に「コントラスト: 未検証」が含まれ、静的指摘が violation として FAIL 判定に数えられる
- **should**: `sb-verify` チェック1 が MCP 検出時に MCP 経由で一覧取得する (React + addon-mcp 案件で確認)
- **should**: 大規模対象 (10ファイル超) で `ds-audit` / `sb-verify` が収集をサブエージェントにファンアウトする
- **could**: 成果物が対象プロジェクト側の `docs/proposals/` / `docs/audit/` に保存される (uidd リポジトリ側に書き込まない)

## 回帰

- **must**: プラグインが `/plugin` 経由で読み込める (frontmatter 破壊がない) — commands の frontmatter は今回非変更
- **must**: CHANGELOG 先頭バージョンと plugin.json version の一致 (0.4.0) — **カバー済み** (目視確認済み)
- **should**: README のコマンド説明 (3チェック独立実行等) が編集後も正確 — **カバー済み** (説明文は変更内容と矛盾なし)
