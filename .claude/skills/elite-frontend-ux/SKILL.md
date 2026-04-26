---
name: elite-frontend-ux
description: WebフロントエンドUIを作るときに常時適用するコアスキル。デザイン哲学・禁止事項・アクセシビリティ必須ルールを提供し、AIっぽい無難なUIを防ぐ。
---

# Elite Frontend UX — Core

**原則:** Bold aesthetic choices + systematic execution = memorable interfaces. Generic is the enemy.

---

## 1. デザイン哲学（コード前に必ず宣言する）

**3つを明示してから進む:**
1. WHO — 誰が、どんな状況で使うか
2. WHAT — ユーザーに取ってほしい唯一のアクション
3. AESTHETIC — 以下から1つ選んでコミットする（曖昧にしない）

| スタイル | 参考 |
|---|---|
| Brutally minimal | Stripe, Linear |
| Playful/toy-like | Figma, Notion |
| Neo-brutalist | 意図的に荒削り、露出感 |
| Organic/natural | 手描き風、テクスチャ |
| Luxury/refined | ファッションブランド |
| Maximalist editorial | Bloomberg, Awwwards |
| Retro-futuristic | Y2K, vaporwave |

**記憶テスト:** ユーザーがこの画面で覚えること1つを言えるか？言えなければ設計が薄い。

---

## 2. デザイントークン（必ず使う値）

**タイポグラフィ:**
- body最小: `text-base` (16px) — モバイルでは絶対に下回らない
- 行の長さ: `max-w-prose` または `max-w-2xl` で45-75文字に制限
- フォント: **Inter / Roboto / Arial を display fontに使わない**（AIの平凡さの象徴）
- 推奨 display font: Fraunces, Space Grotesk, Cabinet Grotesk, Satoshi, Clash Display
- 推奨 body font: IBM Plex Sans, Plus Jakarta Sans, Work Sans

**カラー:**
- 比率: 60% dominant / 30% secondary / 10% accent
- アクセントは1色まで
- **白背景に紫・青グラデーション禁止**（AI cliché）
- ダークモードはHSL CSS変数で制御（`--background`, `--primary` 等）

**アニメーション:**
- ボタンフィードバック: 100-150ms（即時に感じる速さ）
- animate対象: `transform` と `opacity` のみ（GPU accelerated）
- **`width` / `height` / `margin` のアニメーション禁止**（reflow）
- `prefers-reduced-motion` を必ず考慮する

---

## 3. アクセシビリティ（省略不可）

| 要素 | 最低コントラスト比 |
|---|---|
| 本文テキスト | 4.5:1 |
| 大きなテキスト(18pt+) | 3:1 |
| UIコンポーネント・アイコン | 3:1 |
| フォーカスリング | 3:1 |

- タッチターゲット最小: **44×44px**（間隔8px以上）
- フォーカス: `outline: none` の使用禁止（代替なしは不可）
- フォーム: `<label>` 必須、エラーは `aria-describedby` で紐付ける
- アイコンのみのボタン: `aria-label` 必須
- セマンティックHTML優先（ARIAはネイティブHTMLで代替できない場合のみ）

---

## 4. Tailwindの落とし穴（AIが間違えやすい）

```tsx
// ❌ Tailwindがpurgeして動かない
<div className={`bg-${color}-500`} />

// ✅ オブジェクトマップで完全なクラス名を使う
const map = { blue: "bg-blue-500", red: "bg-red-500" }
<div className={map[color]} />
```

- 条件付きクラスは必ず `cn()` (clsx + tailwind-merge) で合成する
- コンポーネントのvariantは CVA (`class-variance-authority`) で管理する
- ダークモード: クラスベース (`.dark`) を優先する

---

## 5. 禁止リスト

**見た目:**
- ❌ 白背景に紫/青グラデーション
- ❌ Inter / Roboto / Arial をdisplay fontに使用
- ❌ border-radiusの不統一（4px/8px/12pxのどれか1つに統一）
- ❌ 光源と合わない影

**UX:**
- ❌ Confirmshaming（「いいえ、節約は嫌いです」）
- ❌ 偽りの緊急性・希少性
- ❌ フォームの送信ボタンを試行前から無効化
- ❌ placeholder をlabelの代わりに使う

**技術:**
- ❌ `<div onclick>` を `<button>` の代わりに使う
- ❌ レイアウトプロパティのアニメーション
- ❌ Dynamic Tailwindクラス名

---

## 6. 納品前チェック

- [ ] body text コントラスト ≥ 4.5:1
- [ ] タッチターゲット ≥ 44×44px
- [ ] `alt` テキストあり（装飾画像は `alt=""`）
- [ ] フォームに `<label>` あり
- [ ] フォーカス状態が見える
- [ ] ローディング・エラー・空状態が実装されている
- [ ] `prefers-reduced-motion` 対応
- [ ] アニメーションは transform/opacity のみ
- [ ] 1ページに主要アクションが1つ
