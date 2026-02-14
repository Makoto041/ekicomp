# ADR-001: 技術スタックの選定

## ステータス
承認済み

## 日付
2026-02-14

## コンテキスト
個人開発の駅コンプリートアプリの技術スタックを選定する必要がある。
開発者はNext.js / TypeScript / Firebaseの経験があり、バックエンドにKotlinを学習目的で採用したい。
ランニングコストは月500円以内に抑えたい。

## 決定

### バックエンド: Kotlin + Ktor
- **理由**: Kotlin学習が主目的。Spring Bootより軽量で、個人開発の規模に合う
- **トレードオフ**: Spring Bootの方がエコシステムは大きいが、学習コストと起動速度の面でKtorを優先

### フロントエンド: Next.js + TypeScript
- **理由**: 使い慣れたフレームワーク。PWA対応が容易。Vercelで無料ホスティング
- **トレードオフ**: 新しいフレームワーク（Remix等）も選択肢にあったが、学習コストをバックエンドに集中させるため見送り

### データベース: Supabase (PostgreSQL)
- **理由**: Auth + DB + Storage が一体。無料枠が大きい（500MB DB, 1GB Storage）。Firebase代替として
- **トレードオフ**: Firebaseの方が経験があるが、PostgreSQLの方がKtorとの相性が良く、RLSも強力

### ホスティング: Vercel (FE) + Railway (BE)
- **理由**: Vercelは無料Hobby枠。RailwayはDockerデプロイが簡単でKotlinとの相性が良い
- **トレードオフ**: Renderも候補だったが、Railwayの方がコールドスタートが速い

## 結果
月額コストは初期ほぼ無料、スケール時も500円以内に収まる見込み。
