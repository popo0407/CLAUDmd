---
name: aws-s3
description: aws S3バケットを構築する。
---
# Instructions
S3バケットの作成・変更時は、以下の最新ベストプラクティスを順に適用せよ。

## 1. セキュリティ (Modern Security)
- **パブリックアクセスの完全遮断**: `Block Public Access` を全項目有効化せよ。
- **ACLの無効化**: `Object Ownership` を `BucketOwnerEnforced` に設定し、ACLを完全に無効化せよ。
- **TLSの強制**: バケットポリシーで `aws:SecureTransport: false` のリクエストを明示的に拒否せよ。
- **暗号化の選択**: デフォルト（SSE-S3）を利用するか、鍵管理の要件がある場合は `SSE-KMS` を設定せよ。

## 2. コストとデータ管理 (Cost & Lifecycle)
- **バージョニング**: 誤削除防止のため有効化せよ。
- **ライフサイクル設定**: 30日以上経過したオブジェクトを `Intelligent-Tiering` へ移行、または旧バージョンを自動削除する設定を検討せよ。

## 3. 命名と整合性 (Naming)
- **命名規則**: [プロジェクト名]-[環境]-[用途]-[AWSプロファイル] とし、すべて小文字・ハイフン繋ぎを徹底せよ。

# Execution Steps
1. **バケット作成**: `aws s3api create-bucket` を実行。
2. **パブリックアクセス遮断**: `put-public-access-block` を適用。
3. **ACL無効化**: `put-bucket-ownership-controls` で `ObjectOwnership=BucketOwnerEnforced` を設定。
4. **暗号化・TLS設定**: `put-bucket-encryption` および `put-bucket-policy` を実行。
5. **ライフサイクル適用**: `put-bucket-lifecycle-configuration` でコスト最適化を自動化。
6. **記録**: `AWS_RESOURCES.md` にバケット名と用途を記録。