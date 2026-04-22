## 使用言語・フレームワーク
- 言語（バックエンド）: Python 3.13
- 言語（フロントエンド）: HTML / JavaScript / CSS（フレームワーク未定、スマホ対応必須）
- フレームワーク（バックエンド）: AWS Lambda（サーバーレス）

## 外部API・サービス連携
| サービス | 用途 |
|---------|------|
| Amazon S3 | 配布資料（PDF/画像）の保存 |
| Amazon Bedrock (Amazon Nova Lite v2) | 資料の内容解析・スケジュール抽出・要約生成・OCR（画像/PDF直接解析）。モデルID: `jp.amazon.nova-2-lite-v1:0` |
| Amazon Cognito | ユーザー認証（保護者ログイン） |
| Amazon API Gateway | REST API エンドポイント |
| Amazon DynamoDB | カレンダーイベント・資料メタデータの保存 |
| Amazon EventBridge | アラート・リマインダーのスケジュール管理 |
| Amazon SNS / SES | メール・プッシュ通知送信（要確認） |
| Amazon CloudFront | フロントエンド配信（低レイテンシ） |

## 依存ライブラリ（バックエンド予定）
- boto3 最新: AWS SDK
- aws-lambda-powertools: Lambda構造化ログ・トレーシング
- pypdf2 / pdf2image: PDFを画像に変換してNova Liteへ渡す前処理（Textract不要）

## 環境情報
- 実行環境: AWS Lambda（Python 3.13 ランタイム）
- ファイルサーバー: Amazon S3
- 開発OS: Windows

## 技術的な制約・注意事項
- **コスト最小化**: Lambda（使用分課金）・DynamoDB（オンデマンド）・S3（低頻度アクセス）を優先
- Nova Lite はマルチモーダル対応のため Textract 不要。PDF は pdf2image で画像変換してからBedrockへ送信
- Bedrock Nova Lite は呼び出し毎課金のため、重複処理を避けるキャッシュ設計が必要（解析済みフラグをDynamoDBに保持）
- Lambda のタイムアウト上限（15分）があるため、AI処理は非同期（S3トリガー + SQS）が望ましい
- スマホブラウザ対応のため、フロントエンドはレスポンシブデザイン必須
- Cognito 無料枠：MAU 50,000 まで無料（個人利用なら問題なし）
