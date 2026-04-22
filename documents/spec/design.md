## 1. アーキテクチャ概要

```
[スマホブラウザ (PWA)]
    │ HTTPS
    ▼
[CloudFront] ──── S3 (静的ホスティング: HTML/JS/CSS)
    │
    │ API呼び出し (JWT付き)
    ▼
[API Gateway (REST)]
    ├── /auth        → Lambda: Cognito User Pool 操作
    ├── /children    → Lambda: 子供プロフィール CRUD
    ├── /upload      → Lambda: S3 署名付きアップロードURL発行
    ├── /documents   → Lambda: 資料メタデータ取得・署名付き閲覧URL発行
    ├── /events      → Lambda: カレンダーイベント CRUD
    ├── /reminders   → Lambda: リマインダー登録 (EventBridge)
    └── /chat        → Lambda: AIチャット (Bedrock)

[S3 バケット: 資料ファイル]
    │ ObjectCreated イベント
    ▼
[Lambda: AI解析ワーカー]
    │ Bedrock (jp.amazon.nova-2-lite-v1:0)
    │ pdf2image で PDF→画像変換 → Nova Lite にマルチモーダル送信
    ▼
[DynamoDB] ← 全データ永続化
    ▼
[EventBridge Scheduler] → Lambda: Push通知送信 (Web Push API)
```

## 2. コンポーネント設計

### DynamoDB テーブル設計（シングルテーブル設計）

| PK | SK | 用途 |
|----|----|------|
| `GROUP#{groupId}` | `USER#{userId}` | 家族グループ・ユーザー |
| `GROUP#{groupId}` | `CHILD#{childId}` | 子供プロフィール |
| `GROUP#{groupId}` | `DOC#{docId}` | 資料メタデータ（S3キー・解析ステータス・要約） |
| `GROUP#{groupId}` | `EVENT#{eventId}` | カレンダーイベント |
| `GROUP#{groupId}` | `REMINDER#{reminderId}` | リマインダー設定 |

- GSI: `childId-date-index`（子供IDと日付でイベントを高速検索）

### Lambda: AI解析ワーカー
- **トリガー**: S3 ObjectCreated
- **入力**: S3オブジェクトキー（パスに groupId・childId[] を埋め込む）
- **処理**: ① PDF→画像変換(pdf2image) → ② Bedrock Nova Lite でOCR+解析 → ③ イベント種別判定+リマインダー候補生成 → ④ DynamoDB保存 → ⑤ 解析済みフラグ更新
- **タイムアウト**: 120秒（NFR-002の30秒は解析完了通知をポーリングで実現）

### Lambda: AIチャット
- **入力**: `{groupId, childIds[], question}`
- **処理**: ① childIds + 質問からキーワードをBedrock抽出 → ② DynamoDB GSI検索 → ③ コンテキスト付きでBedrock回答生成
- **出力**: 回答テキスト（streaming対応推奨）

### フロントエンド構成
- バニラJS + CSS（フレームワーク不使用でバンドルサイズ最小化）
- `manifest.json` + Service Worker でPWA化・プッシュ通知対応
- Cognito Hosted UI は使わず自前ログイン画面（UX統一のため）
- LocalStorage にリフレッシュトークン保存（REQ-005対応）

## 3. 意思決定の記録（ADR）

### ADR-001: フロントエンドフレームワーク選定
- **決定日**: 2026-04-22 / **状態**: 採用済み
- **背景**: スマホ対応PWAが必要。コスト最小化のためCDN配信の静的ファイルのみ。
- **選択肢**:
  | 案 | メリット | デメリット |
  |----|----------|------------|
  | バニラJS（採用） | ゼロ依存・ビルド不要・最速 | 大規模化に不向き |
  | React / Vue | 開発効率高 | ビルド環境・依存管理が必要 |
- **決定理由**: 個人利用・小規模・ビルド環境不要、バニラJSで十分。

### ADR-002: AI解析の同期 vs 非同期
- **決定日**: 2026-04-22 / **状態**: 採用済み
- **背景**: Bedrock呼び出し＋pdf2image変換が30秒超える可能性あり。API GatewayのタイムアウトはHTTP統合で最大29秒。
- **選択肢**:
  | 案 | メリット | デメリット |
  |----|----------|------------|
  | S3トリガー非同期（採用） | タイムアウト回避・コスト低 | フロントがポーリング必要 |
  | 同期Lambda | シンプル | 29秒制限に引っかかるリスク |
- **決定理由**: 安定性優先。フロントは5秒間隔でステータスをポーリングしUXを担保。

### ADR-003: DynamoDB シングルテーブル vs マルチテーブル
- **決定日**: 2026-04-22 / **状態**: 採用済み
- **背景**: 個人利用で読み取りパターンが限定的。コスト最小化が最優先。
- **選択肢**:
  | 案 | メリット | デメリット |
  |----|----------|------------|
  | シングルテーブル（採用） | RCU/WCU共有・低コスト | 設計複雑 |
  | マルチテーブル | 直感的 | テーブル数分のオーバーヘッド |
- **決定理由**: 月150円以下の制約（NFR-003）に対応するためシングルテーブルを採用。

### ADR-004: プッシュ通知方式
- **決定日**: 2026-04-22 / **状態**: 採用済み
- **背景**: SNS/SESは月コスト発生。個人利用なのでサーバーコスト最小化。
- **選択肢**:
  | 案 | メリット | デメリット |
  |----|----------|------------|
  | Web Push API + EventBridge（採用） | 追加コストゼロ | ブラウザ許可が必要・iOS Safari は16.4以降 |
  | Amazon SNS | 確実 | 月数円のコスト増・APN/FCM設定必要 |
- **決定理由**: コスト制約を優先。iOS 16.4以降でWeb Pushが使えるため実用的。

## 4. データフロー

**資料アップロード〜カレンダー登録**
```
保護者 → [子供選択 + ファイル選択]
→ API GW /upload → Lambda → S3署名付きURL発行
→ フロント直接S3アップロード（S3キー: {groupId}/{childIds}/{docId}.ext）
→ S3トリガー → Lambda AI解析ワーカー
→ Bedrock解析 → DynamoDB保存（イベント+リマインダー候補）
→ フロントポーリング → 解析完了通知 → カレンダー更新 + リマインダー提案UI表示
```

**AIチャット**
```
保護者 → [子供選択 + 質問入力]
→ API GW /chat → Lambda
→ Bedrock でキーワード抽出 → DynamoDB GSI検索（childId + 期間）
→ コンテキスト構築 → Bedrock 回答生成 → レスポンス返却
```

## 5. セキュリティ考慮事項
- **認証**: Cognito JWT を全APIで必須検証。リフレッシュトークン有効期限30日（LocalStorage保存）
- **データ分離**: Lambda内で `groupId` は必ずCognitoトークンから取得し、リクエストボディの値を信用しない
- **ファイルアクセス**: S3はパブリックアクセス完全ブロック。署名付きURL（15分）のみ許可
- **個人情報**: 子供の氏名・学年を保存するためDynamoDB暗号化（AWS管理キー）を有効化
- **CORS**: API GatewayのCORSはCloudFrontドメインのみ許可

## 6. 非機能要件への対応
- **NFR-001 (3秒以内)**: CloudFrontキャッシュ + 静的JS/CSSのgzip圧縮
- **NFR-002 (30秒)**: 非同期処理+ポーリングで体感速度を誤魔化さず、実進捗をプログレス表示
- **NFR-003 (150円/月)**: 全サービスを無料枠内に収める設計（月利用量を試算済み）
- **NFR-007 (可用性)**: Lambda + S3 + DynamoDB はすべてマネージドでAWSが可用性保証
- **NFR-008/009 (スマホ対応)**: CSS viewport・flexbox・44pxタッチターゲット標準適用
