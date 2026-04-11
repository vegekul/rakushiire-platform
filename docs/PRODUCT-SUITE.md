# ラクシーレ プロダクトスイート設計書

> 作成日: 2026-04-11
> 管理リポジトリ: vegekul/rakushiire-platform

---

## プロダクトスイートの全体像

ラクシーレは「飲食店の仕入れOS」を目指す3プロダクトのスイート。
それぞれが独立して動作しながら、共通ブランドと相互導線で連携する。

```
┌─────────────────────────────────────────────────────────┐
│                   ラクシーレ エコシステム                    │
│                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐  │
│  │ 飲食店食材相場    │  │ 飲食店食材仕入れ  │  │ 飲食店メニュー    │  │
│  │ チェッカー        │  │ チェッカー        │  │ 開発AI           │  │
│  │                 │  │                 │  │                  │  │
│  │ 食材の今の相場   │→│ 仕入れコストを    │←│ 新メニューの食材  │  │
│  │ をリアルタイム   │  │ AI が最適化      │  │ コストをシミュレ  │  │
│  │ チェック         │  │                 │  │ ート             │  │
│  └─────────────────┘  └─────────────────┘  └──────────────────┘  │
│         ↑                      ↑                    ↑            │
│         └──────────── 共通Google OAuth ──────────────┘            │
│                        共通ナビゲーション                           │
└─────────────────────────────────────────────────────────┘
                              ↓
                    ベジクル取引（自然転換）
```

---

## 各プロダクトの役割

### 1. 飲食店食材相場チェッカー powered by ラクシーレ
- **リポジトリ**: [vegekul/foodmarket-insights](https://github.com/vegekul/foodmarket-insights)
- **コア価値**: 食材の市場相場をリアルタイムで確認できる
- **SEOポテンシャル**: 最高（「食材相場」「野菜相場」は月10K〜100K検索）
- **スイート内役割**: **集客の主軸**。食材相場を調べに来たユーザーを仕入れチェッカーに誘導する
- **ターゲット**: 食材価格高騰に悩む飲食店オーナー全般

### 2. 飲食店食材仕入れチェッカー powered by ラクシーレ
- **リポジトリ**: [vegekul/ai-procurement-optimizer](https://github.com/vegekul/ai-procurement-optimizer)
- **コア価値**: 食べログURL/納品書 → AIが最適な仕入れ先を提案
- **SEOポテンシャル**: 中（「飲食店 仕入れ最適化」1K〜10K）
- **スイート内役割**: **収益化・ベジクル転換の主軸**。最もラクシーレ（ベジクル）との取引に近い
- **本番稼働中**: https://ai-procurement-optimizer-wine.vercel.app

### 3. 飲食店メニュー開発AI powered by ラクシーレ
- **リポジトリ**: [vegekul/trend-food-ai](https://github.com/vegekul/trend-food-ai)
- **コア価値**: AIがトレンドメニューを提案 → レシピ×原価計算 → 企画書PDF出力
- **SEOポテンシャル**: 中（「飲食店 メニュー開発」「新メニュー 考え方」）
- **スイート内役割**: **エンゲージメント強化**。メニュー開発から食材仕入れへの自然な導線
- **ステータス**: 機能再設計フェーズ（[Issue #5](https://github.com/vegekul/rakushiire-platform/issues/5)）

---

## スイート横断技術要件

### 共通ブランディング（[Issue #1](https://github.com/vegekul/rakushiire-platform/issues/1)）

| 要素 | 仕様 | ステータス |
|---|---|---|
| プロダクト名 | `{SEOキーワード} powered by ラクシーレ` | ✅ 仕入れチェッカーで実装済み |
| Primary Color | `#1e40af` (Tailwind: blue-800) | 暫定 |
| Accent Color | `#4338ca` (Tailwind: indigo-700) | 暫定 |
| ロゴ | 🥬 SVG化予定 | 未実装 |
| OGP画像 | 1200×630px | 未実装 |

### 共通認証（[Issue #2](https://github.com/vegekul/rakushiire-platform/issues/2)）

| プロダクト | 現在の認証 | 目標 |
|---|---|---|
| 仕入れチェッカー | NextAuth v5 + Credentials + Google OAuth | ✅ |
| 食材相場チェッカー | Supabase Auth（モックモード） | NextAuth v5 移行 or Supabase Google OAuth 有効化 |
| メニュー開発AI | NextAuth v5 + Supabase + Google OAuth | Google OAuth の Client ID を統一 |

**方針**: 同一 Google Cloud Project の OAuth クライアントを3プロダクトが共有する。
DB は各プロダクト個別管理のまま（User テーブルは分離）。

### 共通ナビゲーション（[Issue #4](https://github.com/vegekul/rakushiire-platform/issues/4)）

全プロダクトのヘッダーに統一ナビを実装:

```
[🥬 ラクシーレ] [現在地プロダクト名] | [相場チェッカー ↗] [仕入れチェッカー ↗] [メニュー開発AI ↗]
```

---

## ユーザージャーニー（スイート横断）

```
[食材相場チェッカー]
  「キャベツの相場を調べる」（SEO流入）
         ↓
  「自分の仕入れ価格と比べてみる」
         ↓
[仕入れチェッカー]
  「納品書をアップして全体を分析」
         ↓
  ダッシュボードでコスト推移を管理
         ↓
[メニュー開発AI]
  「安くなった食材でメニューを考える」
         ↓
  「この食材を安く仕入れたい」→ 仕入れチェッカーへ
         ↓
[ベジクルへの自然転換]
  「ラクシーレ経由でベジクルと取引開始」
```

---

## ポータルサイト構成（[Issue #3](https://github.com/vegekul/rakushiire-platform/issues/3)）

```
rakushiire.jp（予定）
  /                  ← ブランドLP（3プロダクト紹介）
  /cases             ← BeforeAfter 事例集
  /blog              ← コンテンツマーケティング
  /tools             ← 無料計算ツール集（原価率計算 等）
```

---

## 更新履歴

| 日付 | 更新内容 |
|------|---------|
| 2026-04-11 | 初版作成（vegekul/rakushiire-platform 新設に伴い） |
