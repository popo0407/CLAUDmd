---
name: deploy-long-running-api
description: RESTAPIではタイムアウトしてしまう30秒を超える処理に対して最適なパターンで構築する。
---
# Instructions
長時間処理の要件に対し、以下の3つのパターンからユーザーの選択に基づきベストプラクティスを適用せよ。

---

## パターンA：非同期REST (HTTP 202 + ポーリング)
**【用途】シンプルかつ堅牢な処理。進捗のリアルタイム性が不要な場合。**
- **受付**: API Gatewayでリクエストを受け、即座に `202 Accepted` と `jobId` を返す。
- **実行**: LambdaからSQS経由、または直接 Step Functions を起動して非同期実行。
- **確認**: フロントエンドが `jobId` を使ってDynamoDB等のステータスを定期確認（ポーリング）する。
- **ポイント**: API Gatewayの29秒タイムアウトを完全に回避でき、実装が最も単純。

## パターンB：WebSocket API
**【用途】リアルタイム進捗、AIのストリーミング出力、双方向通信が必要な場合。**
- **接続管理**: `$connect`/`$disconnect` で `connectionId` をDynamoDBで管理。
- **プッシュ通知**: 処理完了時、または中間経過を `PostToConnection` でサーバーからプッシュ。
- **タイムアウト対策**: アイドルタイムアウト対策として、アプリ層での「Ping/Pong」による生存確認を実装。

## パターンC：AppSync (GraphQL Subscription)
**【用途】サーバーレスかつマネージドに、データの変更をリアルタイム通知したい場合。**
- **仕組み**: Mutation（データ更新）をトリガーに、Subscription（購読）しているクライアントへ自動通知。
- **メリット**: WebSocketの接続管理（ConnectionIdの保存等）をAWS側が隠蔽してくれるため、開発効率が高い。
- **認証**: OIDCやIAM、API Keyなど、きめ細やかな認可制御を容易に統合。

## ⚠️ アンチパターン回避 (Anti-Patterns)
- **REST**: ポーリングによるコスト増を防ぐため、DynamoDB TTLの設定と、フロント側での指数バックオフ実装を提案せよ。
- **WebSocket**: 410 Gone エラー検知時に DynamoDB から古い connectionId を削除するクリーンアップ処理を必ず実装せよ。
- **AppSync**: 不必要なデータ転送を避けるため、Subscription には必ずフィルタリング用の引数を設けよ。
---

# Execution Steps
1. **要件確認**: ストリーミングの要否、クライアントの接続維持能力に基づき、上記A・B・Cからユーザーに選択を促す。