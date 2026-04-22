---
name: aws-frontend-infra
description: AWSのS3, CloudFront, WAFを組み合わせたフロントエンドを構築する。
---
# Instructions
S3 + CloudFront + WAF の構成を構築する際は、以下のベストプラクティスを遵守せよ。

## 1. 権限と前提条件の確認
- **WAFのリージョン**: CloudFront用のWAFは `` を利用すること。

## 2. セキュリティ・設計 (Security First)
- **S3 (Origin)**: 
    - 「静的ウェブサイトホスティング」機能は**無効**のまま、プライベートバケットとして運用する。
    - **OAC (Origin Access Control)** を使用し、CloudFrontからのアクセスのみを許可する（旧式のOAIは非推奨）。
- **CloudFront**:
    - `Viewer Protocol Policy` は `redirect-to-https` を強制。
    - TLS 1.2以上を使用。

## 3. コストと最適化
- **S3バケットキー**: KMS暗号化コスト削減のため有効化。
- **ライフサイクル**: `AbortIncompleteMultipartUpload` を7日後に設定。

# Execution Steps
1. **S3作成**: バケット作成、パブリックアクセスの完全ブロック。
2. **CloudFront作成**: OACの設定、WAFの紐付け、S3をオリジンに指定。
3. **ポリシー更新**: S3バケットポリシーを更新し、CloudFront OACからの `s3:GetObject` を許可。
4. **ドキュメント更新**: `documents/AWS_RESOURCES.md` にリソースIDを記録。