## 目的
幼稚園・小学校から配布される紙・PDF資料をスマートフォンからアップロードし、
AIが日程・持ち物・お知らせを自動抽出してカレンダー登録・リマインダー送信・内容説明を行う
保護者向けWebアプリケーション。

## ファイル・フォルダ構造（想定）

```
project-root/
├── frontend/                      # スマホ対応Webフロントエンド (HTML/JS/CSS)
│   ├── index.html                 # エントリポイント（PWA対応想定）
│   ├── calendar.html              # カレンダー画面
│   ├── upload.html                # 資料アップロード画面
│   ├── js/                        # JavaScriptモジュール
│   └── css/                       # スタイルシート
├── backend/                       # Python 3.13 Lambda関数群
│   ├── upload_handler/            # S3アップロード受付 + Textract/Bedrock呼び出しトリガー
│   ├── ai_processor/              # AI解析・スケジュール抽出・要約生成
│   ├── calendar_api/              # イベントCRUD API
│   ├── notification/              # アラート送信（EventBridge + SNS/SESなど）
│   └── auth/                      # 認証（Cognito連携）
├── infrastructure/                # IaC（CloudFormation / CDK）
│   └── template.yaml              # AWSリソース定義
└── documents/                     # ドキュメント類
    ├── system_tree.md
    ├── AWS_RESOURCES.md
    └── spec/                      # 仕様駆動開発ドキュメント
```

## 不明点・確認が必要な事項
- IaCツール（CDK / CloudFormation / SAM）は未決定
- 複数の保護者（家族間）でデータを共有するか未確認
- 通知手段（メール / プッシュ通知 / LINE連携）は未確認
- 資料の入力形式（PDF / 画像 / 両方）は未確認
