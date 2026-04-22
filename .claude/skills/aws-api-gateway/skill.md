---
name: aws-api-gateway
description: CloudFront + API Gateway + Lambda のサーバーレスバックエンドを新規構築・設定する際に、セキュリティ・パフォーマンス・運用のベストプラクティスを適用するスキル。
---
# Instructions

## 1. セキュリティ
- エンドポイントタイプは `Regional` を使用し、CloudFront経由のリクエストのみ許可する
- CloudFrontが付与するカスタムヘッダー（`X-Origin-Verify`等）をAWS WAFで検証する
- LambdaへのInvoke権限（`lambda:InvokeFunction`）はソースARNを指定して最小権限で付与する
- CORSはCloudFrontまたはAPI Gatewayのいずれか一方のみで処理し、二重設定を避ける

## 2. Lambda設計
- アーキテクチャは `arm64` を優先（コスト最適化）
- タイムアウトはAPI Gatewayのハード上限（29秒）以下に設定する
- 30秒を超える処理は `.claude/skills/aws-long-time-api/skill.md` の非同期パターンを使用する
- CloudWatch Logsの保持期間は30日、JSON構造化ログで出力する

## 3. API Gateway設定
- Lambdaとの統合は `Lambda Proxy Integration` を使用する
- `MinimumCompressionSize` を1024バイトに設定してペイロード圧縮を有効化する
- メソッドレベルのメトリクスとX-Rayトレースを有効化する
- レート制限はLambdaの予約済み同時実行数に基づいて設定する

## 実行手順
1. **IAM**: Lambda用の実行ロールを作成（信頼ポリシー設定）
2. **Lambda**: `arm64` アーキテクチャでデプロイ、ログ保持期間を30日に設定
3. **API作成**: `aws apigateway create-rest-api --endpoint-configuration types=REGIONAL`
4. **統合設定**: Lambda Proxy Integrationを構成し、呼び出し権限をソースARN指定で付与
5. **WAF設定**: Web ACLを作成し、カスタムヘッダー検証ルールを追加、API Gatewayステージに紐付け
6. **CloudFront**: オリジンにAPI Gatewayを指定し、カスタムヘッダーを設定
7. **デプロイ・記録**: ステージ作成後、`documents/AWS_RESOURCES.md` にリソースIDとエンドポイントを記録
---