---
name: webapp-testing
description: Playwrightを用いてローカルWebアプリケーションのE2Eテストを自動化するスキル。マルチブラウザ対応、フォーム検証、ビジュアルリグレッション、APIモック、スクリーンショット取得などの包括的なテスト機能を提供する。
---

# Instructions

## このスキルの目的
ローカルWebアプリのE2Eテストを**Playwright + pytest**で自動化する。
指定されたURLや操作フローをもとに、即実行可能なテストスクリプトを生成する。

## 技術前提
- **テストフレームワーク:** Playwright (Python)
- **テストランナー:** pytest
- **対象ブラウザ:** Chromium（デフォルト）、Firefox、WebKit
- **Python:** 3.13

## 実行手順

### Step 1: テスト対象の確認
以下を明示してから進む（不明な場合はユーザーに確認）：
1. **対象URL** - テスト対象のベースURL（例: `http://localhost:3000`）
2. **テストシナリオ** - 何を検証したいか
3. **認証有無** - ログインが必要か（ある場合は環境変数で管理）
4. **ブラウザ** - 特定のブラウザ指定があるか

### Step 2: テスト環境チェックリスト
スクリプト出力の前に以下を確認するよう促す：
```
事前確認:
- [ ] `pip install playwright pytest-playwright` 実行済み
- [ ] `playwright install chromium` 実行済み
- [ ] テスト対象サーバーが起動済み（または起動スクリプトを用意）
- [ ] 認証情報は `.env` で管理し、ハードコードしない
```

### Step 3: テストコード生成

**ファイル構成:**
```
tests/
  conftest.py        # 共通フィクスチャ（ブラウザ設定、認証等）
  test_[機能名].py   # テストファイル
```

**必須コードパターン:**

```python
# conftest.py
import pytest
from playwright.sync_api import Playwright, BrowserContext
import os

@pytest.fixture(scope="session")
def base_url():
    return os.environ.get("BASE_URL", "http://localhost:3000")

@pytest.fixture
def page(playwright: Playwright):
    browser = playwright.chromium.launch(headless=True)
    context = browser.new_context()
    page = context.new_page()
    yield page
    context.close()
    browser.close()
```

**テストコード品質基準:**
- `page.wait_for_load_state("networkidle")` で非同期リクエスト完了を待つ
- ハードコードの `time.sleep()` は使わない - `expect()` の待機を活用する
- セレクタは優先順位順: `data-testid` > `aria-label` > テキスト > CSSセレクタ
- アサーションは `expect()` を使い、具体的なエラーメッセージが出るようにする
- 認証情報は必ず環境変数で管理する（`.env` + `python-dotenv`）

**テストシナリオ別テンプレート:**

*フォーム検証:*
```python
def test_form_validation(page, base_url):
    page.goto(f"{base_url}/login")
    page.wait_for_load_state("networkidle")
    
    # 空送信の検証
    page.click('button[type="submit"]')
    expect(page.locator("[data-testid='error-message']")).to_be_visible()
    
    # 正常入力の検証
    page.fill('input[name="email"]', os.environ["TEST_EMAIL"])
    page.fill('input[name="password"]', os.environ["TEST_PASSWORD"])
    page.click('button[type="submit"]')
    expect(page).to_have_url(f"{base_url}/dashboard")
```

*スクリーンショット取得（ビジュアルリグレッション）:*
```python
def test_visual_regression(page, base_url):
    page.goto(f"{base_url}/")
    page.wait_for_load_state("networkidle")
    page.screenshot(path="tests/screenshots/home.png", full_page=True)
```

*APIモック（ネットワーク差替え）:*
```python
def test_with_mock_api(page, base_url):
    page.route("**/api/users", lambda route: route.fulfill(
        status=200,
        content_type="application/json",
        body='[{"id": 1, "name": "テストユーザー"}]'
    ))
    page.goto(f"{base_url}/users")
    expect(page.locator("text=テストユーザー")).to_be_visible()
```

### Step 4: 実行コマンドを出力する
```bash
# 基本実行
pytest tests/ -v

# 特定ブラウザで実行
pytest tests/ --browser firefox -v

# レポート付き
pytest tests/ -v --html=report.html

# CI向け（ヘッドレス）
pytest tests/ -v --headless
```

## セキュリティ要件（必須）
- [ ] 認証情報（パスワード、APIキー）をコードにハードコードしない
- [ ] テストは本番環境ではなくステージング/開発環境で実行する
- [ ] `page.route()` でモックするURLを意図的に制限する
- [ ] スクリーンショットに機密情報が映っていないか確認する
- [ ] テストログに個人情報を出力しない

## 注意事項
- 外部コマンド実行を伴うため、スクリプト内容を必ずレビューすること
- テストはCIパイプライン（GitHub Actions等）に組み込み継続実行すること
- UI変更があった場合はテストを速やかに更新すること
