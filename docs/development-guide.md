# EKICOMP 開発フローガイド

## 開発サイクル

```
Issue作成 → ブランチ作成 → 開発（Claude Code併用）→ 動作確認 → PR作成 → セルフレビュー → マージ → Issue閉じる
```

---

## 1. Issue管理

### Issueの粒度
1つのIssueは「1つのPRで完結する」サイズにする。

| 粒度 | 例 | 判定 |
|------|------|------|
| 大きすぎ | 「バックエンド全部作る」 | ❌ 分割する |
| ちょうどいい | 「駅一覧APIを実装する」 | ✅ |
| 細かすぎ | 「変数名をリネーム」 | ❌ 他のIssueに含める |

### ラベル

| ラベル | 用途 |
|--------|------|
| `setup` | 環境構築・初期設定 |
| `feature` | 新機能の追加 |
| `docs` | ドキュメント更新 |
| `bug` | バグ修正 |
| `refactor` | リファクタリング |

---

## 2. ブランチ運用

```
main（本番相当。常に動く状態をキープ）
 ├── feature/station-api       （機能追加）
 ├── feature/map-display        （機能追加）
 ├── fix/checkin-validation     （バグ修正）
 └── docs/update-api-spec      （ドキュメント更新）
```

### ルール
- mainに直接pushしない
- ブランチ名は `feature/○○`、`fix/○○`、`docs/○○` で統一
- マージ後のブランチは削除する

### コマンド例

```bash
# ブランチ作成
git checkout -b feature/station-api

# 開発 → コミット
git add .
git commit -m "feat: 駅一覧API実装"

# push
git push -u origin feature/station-api

# GitHub上でPR作成 → マージ

# mainに戻ってpull
git checkout main
git pull origin main

# マージ済みブランチ削除
git branch -d feature/station-api
```

---

## 3. コミットメッセージ規約

Conventional Commitsに準拠する：

```
<type>: <概要>
```

| type | 用途 | 例 |
|------|------|------|
| `feat` | 新機能 | `feat: チェックインAPI実装` |
| `fix` | バグ修正 | `fix: 訪問日時のタイムゾーン修正` |
| `docs` | ドキュメント | `docs: API仕様書にフィルタパラメータ追加` |
| `refactor` | リファクタ | `refactor: 駅リポジトリの共通化` |
| `chore` | 雑務 | `chore: 依存ライブラリ更新` |
| `test` | テスト | `test: 駅検索APIのテスト追加` |

---

## 4. Claude Code 併用ルール

### 基本方針
AIに全部任せるのではなく、**AIに書かせて→自分で理解して→自分で少し変える**。

### 推奨フロー

```
① タスクを整理する
   「駅一覧APIを作る。GET /stations でDBから全駅を返す」

② Claude Codeに依頼する
   「docs/api-spec.md の GET /stations を参照して、Ktorで実装して」
   → 仕様書をリポジトリに入れてるのでそのまま参照指示できる

③ 生成コードを読む ← 学習ポイント
   - 何をimportしてるか
   - ルーティングの書き方
   - DBアクセスのパターン

④ 自分で少し変えてみる
   - エラーハンドリングを追加
   - フィルタ条件を自分で書く
   - 変数名を自分の好みに変える

⑤ わからなかったことを調べる or メモする
```

### やっていいこと
- ボイラープレート（設定ファイル、雛形）はAIに任せる
- エラーメッセージで詰まったらAIに聞く
- リファクタリングの提案をもらう

### 意識すること
- AIが出したコードをそのまま貼らない。最低限1行ずつ読む
- 「なぜこう書いてるのか」がわからない行はコメントを入れる or 調べる
- PRのdescriptionに学んだことを書く

---

## 5. PR（Pull Request）テンプレート

PRを作る際に以下を書く（雑でOK）：

```markdown
## やったこと
- （このPRで実装した内容）

## 学んだこと
- （開発中に理解したこと、新しく知ったこと）

## 動作確認
- [ ] ローカルで動作確認した
- [ ] エラーケースも確認した

## 残課題
- （このPRでやらなかったこと、別Issueにすべきこと）
```

---

## 6. 動作確認

### Phase 1ではシンプルに
- APIの確認：curlまたはThunder Client（VS Code拡張）
- フロントの確認：ブラウザで目視

### curlの例

```bash
# 駅一覧取得
curl -H "Authorization: Bearer <token>" \
  http://localhost:8080/v1/stations

# チェックイン
curl -X POST \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"stationId": "xxx", "visitedAt": "2026-02-14T18:00:00Z"}' \
  http://localhost:8080/v1/checkins
```

### 自動テスト（余裕があれば）
- バックエンド：Ktor のテスト機能（testApplication）
- フロントエンド：後回しでOK

---

## 7. リリースフロー（将来）

Phase 4以降で一般公開する際に整備する項目：

- ステージング環境の構築
- CI/CD（GitHub Actions）
- 自動テストの拡充
- バージョニング（Semantic Versioning）
- CHANGELOG管理