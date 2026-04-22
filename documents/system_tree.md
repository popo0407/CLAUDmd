# システム構成図
最終更新: 2026-04-22

## 1. ファイル・フォルダ構造

```
CLAUDECODE/                          # プロジェクトルート
├── report.md                        # 仕様駆動開発の調査報告書
└── documents/                       # ドキュメント類
    ├── system_tree.md               # システム構成図（本ファイル）
    ├── AWS_RESOURCES.md             # AWSリソース一覧
    └── spec/                        # 仕様駆動開発ドキュメント
        ├── structure.md             # フォルダ構造・役割定義
        ├── tech.md                  # 使用技術・AWSサービス一覧
        ├── product.md               # プロジェクト概要・スコープ・成功基準
        ├── requirements.md          # EARS記法による機能・非機能要件定義
        ├── design.md                # アーキテクチャ設計・ADR・データフロー
        └── tasks.md                 # 実装タスクリスト（22件・Phase 1〜5）
```

---

## 2. 仕様駆動開発（SDD）フロー

```
[ユーザー: 「作って」]
    --> [sdd-spec] 環境解析 + ヒアリング
        --> documents/spec/structure.md
        --> documents/spec/tech.md
        --> documents/spec/product.md
    --> [sdd-requirements] EARS記法で要件定義
        --> documents/spec/requirements.md
    --> [sdd-design] 技術設計 + ADR記録
        --> documents/spec/design.md
    --> [sdd-tasks] タスク分解 + 優先順位付け
        --> documents/spec/tasks.md
    --> [実装] タスク順に実装・テスト
    --> [sdd-walkthrough] 実装記録・要件カバレッジ確認
        --> documents/spec/walkthrough.md
```

---

## 3. 更新タイミング

- 新規ファイル・フォルダを作成したとき
- 既存ファイルを移動・削除・リネームしたとき
- 大きな機能変更によりデータフローが変わったとき
