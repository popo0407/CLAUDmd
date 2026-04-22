---
name: aws-resource
description: AWSのリソースを構築する。
---
# Instructions
AWSのリソース作成を行う際は、以下の手順を厳守せよ。

## 1. 権限 (Security)
- **最小権限の原則**: リソースに付与するIAMポリシーは、ワイルドカードを避け、必要なアクションとリソースARNを特定して記述せよ。

## 2. 命名規則 (Naming)
- **命名規則**: [プロジェクト名]-[環境]-[用途] とし、すべて小文字・ハイフン繋ぎを徹底せよ。(s3バケットを除く)

## 3. AWS CLIによる実行 (Execution)
- **CLIの優先**: `aws cli` でAIが作成する。ユーザーはリソース作成の操作はしない。

## 4. コスト管理と識別 (Cost Management)
- **タグ付け**: 全てのリソースに[プロジェクト名]のタグを付与する。

## 5. リソース情報の記録 (Documentation & Tracking)
- **自動記録**: 作成完了直後、`documents/AWS_RESOURCES.md`に追記せよ。
- **確認**: 作成したリソースのステータスが `available` または `running` になっていることをCLIで確認し、その結果を報告せよ。

# Execution Steps
1. AWS CLI実行後、戻り値のJSONから重要データを抽出。
2. `documents/AWS_RESOURCES.md` を更新し、完了報告を行う。