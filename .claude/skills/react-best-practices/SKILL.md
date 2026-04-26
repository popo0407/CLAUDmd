---
name: react-best-practices
description: React/Next.jsのコーディングをするときのスキル。ルールに基づいてコードレビュー・改善提案を行う。
---

# Instructions

## このスキルの目的
React/Next.js コードに潜むパフォーマンスリスク・アンチパタ ーンを検出し、
**悪い例と良い例のセット**で具体的な改善策を提示する。
コードを直接書き換えるのではなく、「改善リスト＋実例コード」の形で提案する。

## 実行手順

### Step 1: 対象コードの確認
ユーザーから渡されたコード（またはファイル）を読み込み、以下の観点でスキャンする：
1. **データフェッチ戦略** - クライアント/サーバーのフェッチ分離
2. **レンダリング戦略** - CSR/SSR/SSG/ISRの適切な使用
3. **バンドルサイズ** - 不要な依存関係・動的インポートの欠如
4. **再レンダリング** - 不要な再レンダリングを引き起こす実装
5. **画像・フォント** - 最適化されていないリソース読み込み

### Step 2: 問題点を重要度付きで列挙

出力フォーマット：
```
## パフォーマンス診断: [ファイル名 / コンポーネント名]

### 🔴 Critical（即修正）
### 🟡 Warning（改善推奨）  
### 🔵 Info（余裕があれば対応）
```

### Step 3: 各問題に悪い例・良い例を示す

---

## ルール集（40+ ルール）

### データフェッチ

**[F-1] 🔴 シリアルフェッチをPromise.allで並列化**
```tsx
// ❌ 悪い例: ウォーターフォール発生
const user = await fetchUser(id)
const posts = await fetchPosts(id)  // userを待ってから開始

// ✅ 良い例: 並列実行
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)])
```

**[F-2] 🔴 クライアントフェッチをサーバーコンポーネントに移動**
```tsx
// ❌ 悪い例: useEffectでフェッチ（ウォーターフォール・クライアントバンドル増加）
"use client"
function UserProfile() {
  const [user, setUser] = useState(null)
  useEffect(() => { fetch('/api/user').then(...) }, [])
}

// ✅ 良い例: サーバーコンポーネントで直接フェッチ
async function UserProfile() {
  const user = await fetchUser() // サーバー側で実行
  return <div>{user.name}</div>
}
```

**[F-3] 🟡 React Queryでクライアントフェッチをキャッシュ**
```tsx
// ❌ 悪い例: キャッシュなしで毎回フェッチ
useEffect(() => { fetch('/api/data').then(setData) }, [])

// ✅ 良い例: React Query でキャッシュ管理
const { data } = useQuery({ queryKey: ['data'], queryFn: fetchData })
```

### レンダリング戦略

**[R-1] 🔴 動的データに SSG を使わない**
```tsx
// ❌ 悪い例: 頻繁に更新されるデータに静的生成
export async function generateStaticParams() { ... } // ユーザーごとのダッシュボードには不適切

// ✅ 良い例: 動的データには SSR またはクライアントフェッチ
export const dynamic = 'force-dynamic'
```

**[R-2] 🟡 Suspenseでストリーミングレンダリングを活用**
```tsx
// ❌ 悪い例: ページ全体がデータ待ちでブロック
export default async function Page() {
  const data = await fetchSlowData() // ページ全体がブロック
  return <Component data={data} />
}

// ✅ 良い例: Suspenseで段階的に表示
export default function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <SlowComponent />  {/* 非同期に解決 */}
    </Suspense>
  )
}
```

### バンドルサイズ

**[B-1] 🔴 大きなライブラリを動的インポートで遅延読み込み**
```tsx
// ❌ 悪い例: 重いライブラリを静的インポート
import Chart from 'chart.js'

// ✅ 良い例: 動的インポートで必要時のみ読み込み
const Chart = dynamic(() => import('chart.js'), { ssr: false })
```

**[B-2] 🟡 named importでツリーシェイキングを活用**
```tsx
// ❌ 悪い例: ライブラリ全体をインポート
import _ from 'lodash'
const result = _.debounce(fn, 300)

// ✅ 良い例: 必要な関数のみインポート
import debounce from 'lodash/debounce'
```

**[B-3] 🔵 next/bundleAnalyzerでバンドルを可視化**
```bash
# package.json に追加
ANALYZE=true next build
```

### 再レンダリング防止

**[RE-1] 🔴 useCallbackで関数の参照を安定させる**
```tsx
// ❌ 悪い例: 毎レンダリングで新しい関数が作られる
function Parent() {
  const handleClick = () => { ... }  // 毎回新しい参照
  return <Child onClick={handleClick} />
}

// ✅ 良い例: useCallbackで参照を固定
const handleClick = useCallback(() => { ... }, [依存配列])
```

**[RE-2] 🟡 useMemoで高コスト計算をメモ化**
```tsx
// ❌ 悪い例: 毎レンダリングで重い計算
const result = expensiveCalculation(data)

// ✅ 良い例: dataが変わったときだけ再計算
const result = useMemo(() => expensiveCalculation(data), [data])
```

**[RE-3] 🟡 useEffectの依存配列を正確に管理**
```tsx
// ❌ 悪い例: 依存配列の省略（無限ループや古い値の参照）
useEffect(() => { fetchData(userId) })  // 毎レンダリングで実行

// ✅ 良い例: 依存配列を明示
useEffect(() => { fetchData(userId) }, [userId])
```

**[RE-4] 🔵 Context の分割で不要な再レンダリングを防ぐ**
```tsx
// ❌ 悪い例: 1つの巨大なContextで全状態を管理
const AppContext = createContext({ user, theme, cart, ... })

// ✅ 良い例: 関連する状態ごとにContextを分割
const UserContext = createContext({ user })
const ThemeContext = createContext({ theme })
```

### 画像・フォント・リソース

**[I-1] 🔴 `<img>` を `next/image` の `<Image>` に置き換える**
```tsx
// ❌ 悪い例: 最適化なし
<img src="/hero.png" alt="ヒーロー画像" />

// ✅ 良い例: 自動最適化・遅延読み込み
import Image from 'next/image'
<Image src="/hero.png" alt="ヒーロー画像" width={800} height={400} priority />
```

**[I-2] 🟡 フォントを next/font でセルフホスト**
```tsx
// ❌ 悪い例: 外部Google Fontsを直接読み込み（レイアウトシフト発生）
<link href="https://fonts.googleapis.com/css2?family=Inter" rel="stylesheet" />

// ✅ 良い例: next/fontでセルフホスト
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] })
```

---

## 診断結果フォーマット

```markdown
## パフォーマンス診断結果

### 🔴 Critical（[N]件）
- [F-1] シリアルフェッチ: `fetchUser()` と `fetchPosts()` を並列化してください

### 🟡 Warning（[N]件）
- [RE-2] `expensiveFilter()` を `useMemo` でメモ化してください

### 🔵 Info（[N]件）
- [B-3] バンドルアナライザーで全体サイズを確認することを推奨します

### 修正が不要な箇所
- [I-1] `next/image` は適切に使用されています ✅
```

## 注意事項
- 提案はプロジェクト固有のアーキテクチャ判断の後に採用すること
- すべての変更にはテストを追加し、パフォーマンス計測で効果を確認すること
- トレードオフ（可読性 vs パフォーマンス）がある場合は必ず言及すること
- ルールは `vercel-labs/agent-skills` リポジトリで随時更新されるため、定期的に参照すること
