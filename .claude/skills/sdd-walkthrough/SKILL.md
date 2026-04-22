---
name: sdd-walkthrough
description: 実装フェーズ完了後にAntigravityスタイルのウォークスルー（walkthrough.md）を自動生成する。何をどの順で実装し、どのテストが通り、どんな決定をしたかを人間が追跡できる記録として残す。実装の最後に呼び出すこと。
---

# Instructions

## 前提
- `documents/spec/tasks.md` が存在し、全タスクが完了済みであること
- `documents/spec/requirements.md`・`documents/spec/design.md` を参照する
- 出力先: `documents/spec/walkthrough.md`

## Step 1: 実装結果の収集

以下の情報をすべて収集する：

1. **完了タスク一覧**（tasks.md の全 TASK-ID と完了ステータス）
2. **テスト結果**（各タスクのテスト方法と PASS / FAIL の結果）
3. **実装中の意思決定ログ**（設計から変更した点・追加判断した点）
4. **最終的なファイル構成**（新規作成・変更されたファイル一覧）

## Step 2: walkthrough.md の生成

`.claude/rules/spec-walkthrough.md` のフォーマットに従って生成する。

## Step 3: 完了メッセージの表示

```
✅ ウォークスルー生成完了

documents/spec/walkthrough.md を作成しました。

確認事項:
- 要件カバレッジ: X / X
- 未解決の特記事項: X件
- design.md への ADR 追記が必要: あり / なし

問題がなければ system_tree.md を更新してください。
```

## 注意事項
- テスト結果は「実行した事実」のみ記載し、推測を書かない
- 設計から変えた箇所は必ず理由とともに記録する（後からの修正でAIが誤って戻すことを防ぐ）
- ウォークスルーは **100〜150行以内** に収める。超える場合は要件カバレッジ表を別ファイル（`walkthrough_coverage.md`）に分離する
