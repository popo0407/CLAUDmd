---
name: aws-bedrock-model
description: AWS Bedrock モデルを選択する際に適用するスキル。
---
# Instructions
AWS Bedrock でモデルを選択する際は、以下の基準を順に適用せよ。

## 1. モデルの特性と要件のマッチング
- claude-haiku-4.5: 計量モデルで、コストを抑えつつも高品質なテキスト生成が必要な場合に最適。
- claude-sonnet-4.6：高精細なテキスト生成が求められる場合に適しているが、コストはやや高め。
- claude-opas-4.6：特定のタスク（例：コード生成、複雑な推論）が必要な場合に選択肢となるが、一般的なテキスト生成には過剰な性能を持つ可能性がある。

## 2. モデルID
- claude-haiku-4.5: `us.anthropic.claude-haiku-4.5`
- claude-sonnet-4.6: `us.anthropic.claude-sonnet-4.6`
- claude-opas-4.6: `us.anthropic.claude-opas-4.6`

## 3. 情報更新 
- AWS Bedrockのモデルラインナップは定期的に更新されるため、モデルの使用時にエラーが発生した場合はモデルIDを調べなおしてこのドキュメントを修正すること。
