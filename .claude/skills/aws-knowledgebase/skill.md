---
name: aws-knowledgebase
description: Amazon Bedrock Knowledge Bases を使った RAG 構成システムを構築する
---
# Instructions
このスキルは **Amazon Bedrock Knowledge Bases（以下 KB）を使って RAG 構成を作る際のベストプラクティス**を、実装可能な形で整理するためのもの。
特に以下の前提を必須条件として採用する：

* **S3 Vector（S3Vectors）必須**
* **本文はノンフィルタブル**として扱い、検索・絞り込みに必要な情報は **metadata.json 側に寄せる**
* **CLI 作成フロー**は「S3 Vector とインデックス作成 → KB 作成」の順番を守る

---

### 2. 本文をノンフィルタブルにする（重要な設計思想）

KB は デフォルトでRAG 検索を行う際に「本文全文」をメタデータとして扱い、フィルタ条件に利用できる領域が制限されるため本文（コンテンツ本文）は **フィルタ対象から外す（non-filterable）**前提にして、検索の精度・絞り込みは以下で実現する：

* **metadata.json のキーを増やす**
* 検索や運用で使う属性は metadata に寄せる

---

## metadata.json 設計ベストプラクティス（本文ノンフィルタブル前提）

### 3. metadata.json を「運用用インデックス」として扱う

本文をノンフィルタブルにするなら、metadata.json は単なる付加情報ではなく、**検索を成立させるためのインデックス定義**になる。

metadata.json の責務はユーザーと協議

---

### 4. metadata.json 推奨スキーマポイント

* `visibility` / `access_level` はアクセス制御の未来対応に効く
* フィルターには許可された語彙のみ使う（辞書化）
* 日付は ISO-8601 統一
* metadata にどこから引用したか」が追跡可能になる情報を入れる。
---

## ドキュメント格納方式のベストプラクティス（S3）

### 6. S3 の推奨ディレクトリ設計

KB のデータソースが S3 の場合、構造を最初に固めるべき。

推奨：

```
s3://kb-documents/
  payment/
    requirements/
      REQ-00123/
        body.md
        metadata.json
      REQ-00124/
        body.md
        metadata.json
  hr/
    policies/
      POL-0001/
        body.md
        metadata.json
```

**狙い**

* system/domain/doc_type で物理的に分離
* 事故が起きても削除範囲が限定される
* ライフサイクルポリシーが適用しやすい

---

## chunking について
Bedrock KB の chunking サイズはディフォルトとせずにユーザーと協議する。

---

## 検索（Retrieval）ベストプラクティス

**設計方針**

* ベクトル検索 = 本文の意味検索
* metadata = スコープ制御と精度担保

---

## CLI 作成フロー（必須順序）

### 11. CLI は「S3Vector → Index → KB」の順が必須

運用事故の典型は、KB 作成後にベクトルストア設計を変えたくなること。
この場合 KB を作り直すことになるため、最初から以下を守る：

1. **S3 Vector を作る**
2. **Index を作る**
3. **Knowledge Base を作る**
4. **DataSource を紐づける**
5. **Ingestion Job を流す**

---

### 12. CLI 作成の基本思想（IaC化のため）

CLI 実行時は必ず以下を満たす：

* 作成するリソース名は固定命名規則に従う
* ARN / ID を出力して保存する（後続の create に使う）
* KnowledgeBase は「後から作り直せる」ものとして扱う

---

## 命名規則（推奨）

### 13. 命名は必ず3階層を含める

推奨フォーマット：

* S3Vector: `[プロジェクト名]-[環境]-[用途]`
* Index: `[プロジェクト名]-[環境]-[用途]`
* KnowledgeBase: `[プロジェクト名]-[環境]-[用途]`

---

## 運用・再構築のベストプラクティス

### 14. 「KB は破棄できる」前提で設計する

KB は設定・モデル・chunking・embedding が変わると作り直しが発生する。
そのため以下を守る：

* データ本体は S3 に置く
* ベクトルは S3Vector 側で管理する
* KB は再作成できる状態にする

---

## セキュリティのベストプラクティス（最低限）

### 16. IAM は最小権限で分離

推奨：

* Ingestion 用ロール
* Query 用ロール
* 管理者用ロール

特に ingestion は S3 読み取りが必要なので、権限漏れの温床になる。

---

## 推奨チェックリスト（実装前に確認すること）

* [ ] S3Vector を先に作成した
* [ ] Index を先に作成した
* [ ] KB は後から作り直せる設計になっている
* [ ] 本文はフィルタ対象にしない方針が固まっている
* [ ] metadata.json に system/domain/doc_type を必ず持たせた
* [ ] metadata は enum 化（辞書化）されている
* [ ] doc_id を採番・一意化している
* [ ] schema_version を metadata に入れている
* [ ] ingestion / query の IAM を分離した
* [ ] S3 のパス設計が system/doc_type 単位で分かれている