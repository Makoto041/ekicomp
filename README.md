# EKICOMP 🚉

東京都内の全駅を巡ってコンプリートを目指すWebアプリ。

「行ったことない駅に降りて、街を歩いて、居酒屋に入る」——そんな日常の冒険を記録し、可視化し、共有できる。

## 技術スタック

| レイヤー | 技術 |
|----------|------|
| Frontend | Next.js (TypeScript) / Tailwind CSS / Mapbox GL JS |
| Backend | Kotlin (Ktor) |
| Database | Supabase (PostgreSQL) |
| Hosting | Vercel (Frontend) / Railway (Backend) |
| Auth | Supabase Auth (Google / Apple) |

## プロジェクト構成

```
ekicomp/
├── docs/                  # 仕様書・設計ドキュメント
│   ├── requirements.md    # 要件定義書
│   ├── roadmap.md         # ロードマップ
│   ├── api-spec.md        # API仕様書 (OpenAPI)
│   ├── db-design.md       # DB設計書 (ER図)
│   ├── screen-transition.md # 画面遷移図
│   └── decisions/         # ADR (Architecture Decision Records)
├── backend/               # Kotlin (Ktor) バックエンド
├── frontend/              # Next.js フロントエンド
└── README.md
```

## 開発ロードマップ

| Phase | 概要 | 状態 |
|-------|------|------|
| Phase 1 | MVP — チェックイン + 地図表示 | 🔧 開発中 |
| Phase 2 | メモ・写真・スポット記録 | ⏳ 未着手 |
| Phase 3 | バッジ・ランダム提案 | ⏳ 未着手 |
| Phase 4 | ソーシャル機能 | ⏳ 未着手 |
| Phase 5 | パートナー機能 & 一般公開 | ⏳ 未着手 |

詳細は [docs/roadmap.md](docs/roadmap.md) を参照。

## セットアップ

```bash
# バックエンド
cd backend
./gradlew run

# フロントエンド
cd frontend
npm install
npm run dev
```

## ドキュメント

- [要件定義書](docs/requirements.md)
- [API仕様書](docs/api-spec.md)
- [DB設計書](docs/db-design.md)
- [画面遷移図](docs/screen-transition.md)
- [ロードマップ](docs/roadmap.md)
