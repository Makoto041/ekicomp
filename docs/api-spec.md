# EKICOMP API仕様書（Phase 1）

## 概要

| 項目 | 内容 |
|------|------|
| ベースURL | `https://api.ekicomp.app/v1` |
| 認証方式 | Bearer Token（Supabase Auth JWT） |
| レスポンス形式 | JSON |
| 文字コード | UTF-8 |

---

## 認証

すべてのAPIリクエスト（認証系を除く）にはAuthorizationヘッダが必要：

```
Authorization: Bearer <supabase_access_token>
```

---

## OpenAPI仕様

```yaml
openapi: 3.0.3
info:
  title: EKICOMP API
  description: 駅コンプリートアプリ Phase 1 API
  version: 1.0.0

servers:
  - url: https://api.ekicomp.app/v1
    description: Production
  - url: http://localhost:8080/v1
    description: Local development

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    Station:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          example: "渋谷"
        nameKana:
          type: string
          example: "シブヤ"
        latitude:
          type: number
          format: double
          example: 35.6580
        longitude:
          type: number
          format: double
          example: 139.7016
        lines:
          type: array
          items:
            $ref: '#/components/schemas/LineSummary'
        isVisited:
          type: boolean
          description: "ログインユーザーが訪問済みか"

    LineSummary:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          example: "山手線"
        company:
          type: string
          example: "JR東日本"
        colorHex:
          type: string
          example: "#80C241"

    Line:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        company:
          type: string
        colorHex:
          type: string
        stations:
          type: array
          items:
            $ref: '#/components/schemas/StationSummary'
        progress:
          $ref: '#/components/schemas/LineProgress'

    StationSummary:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        nameKana:
          type: string
        isVisited:
          type: boolean
        stationOrder:
          type: integer

    LineProgress:
      type: object
      properties:
        totalStations:
          type: integer
        visitedStations:
          type: integer
        percentage:
          type: number
          format: float

    Checkin:
      type: object
      properties:
        id:
          type: string
          format: uuid
        stationId:
          type: string
          format: uuid
        stationName:
          type: string
        visitedAt:
          type: string
          format: date-time
        createdAt:
          type: string
          format: date-time

    CheckinRequest:
      type: object
      required:
        - stationId
      properties:
        stationId:
          type: string
          format: uuid
        visitedAt:
          type: string
          format: date-time
          description: "省略時は現在日時"

    UserProfile:
      type: object
      properties:
        id:
          type: string
          format: uuid
        nickname:
          type: string
        avatarUrl:
          type: string
          nullable: true
        totalVisited:
          type: integer
        totalStations:
          type: integer

    UserProfileUpdateRequest:
      type: object
      properties:
        nickname:
          type: string
          maxLength: 50

    OverallProgress:
      type: object
      properties:
        totalStations:
          type: integer
          example: 487
        visitedStations:
          type: integer
          example: 23
        percentage:
          type: number
          format: float
          example: 4.7
        lineProgress:
          type: array
          items:
            type: object
            properties:
              lineId:
                type: string
                format: uuid
              lineName:
                type: string
              company:
                type: string
              colorHex:
                type: string
              totalStations:
                type: integer
              visitedStations:
                type: integer
              percentage:
                type: number
                format: float

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string

security:
  - BearerAuth: []

paths:
  # ============================================
  # 駅（Stations）
  # ============================================
  /stations:
    get:
      summary: 全駅一覧取得
      description: |
        東京都内の全駅を返す。地図表示用。
        ログインユーザーの訪問ステータス付き。
      tags: [Stations]
      parameters:
        - name: lineId
          in: query
          schema:
            type: string
            format: uuid
          description: "路線IDでフィルタ"
        - name: visited
          in: query
          schema:
            type: boolean
          description: "訪問済み/未訪問でフィルタ"
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  stations:
                    type: array
                    items:
                      $ref: '#/components/schemas/Station'
                  total:
                    type: integer

  /stations/search:
    get:
      summary: 駅名検索
      description: 駅名またはカナで部分一致検索
      tags: [Stations]
      parameters:
        - name: q
          in: query
          required: true
          schema:
            type: string
            minLength: 1
          description: "検索クエリ（駅名 or カナ）"
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 50
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  stations:
                    type: array
                    items:
                      $ref: '#/components/schemas/Station'

  /stations/{stationId}:
    get:
      summary: 駅詳細取得
      tags: [Stations]
      parameters:
        - name: stationId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Station'
        '404':
          description: 駅が見つからない
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  # ============================================
  # 路線（Lines）
  # ============================================
  /lines:
    get:
      summary: 全路線一覧取得
      description: 路線ごとの進捗情報付き
      tags: [Lines]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  lines:
                    type: array
                    items:
                      $ref: '#/components/schemas/Line'

  /lines/{lineId}:
    get:
      summary: 路線詳細取得
      description: 路線に属する駅一覧（訪問ステータス付き）
      tags: [Lines]
      parameters:
        - name: lineId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Line'

  # ============================================
  # チェックイン（Checkins）
  # ============================================
  /checkins:
    get:
      summary: チェックイン履歴取得
      description: ログインユーザーのチェックイン履歴（新しい順）
      tags: [Checkins]
      parameters:
        - name: stationId
          in: query
          schema:
            type: string
            format: uuid
          description: "駅IDでフィルタ"
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
        - name: offset
          in: query
          schema:
            type: integer
            default: 0
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  checkins:
                    type: array
                    items:
                      $ref: '#/components/schemas/Checkin'
                  total:
                    type: integer

    post:
      summary: チェックイン登録
      tags: [Checkins]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CheckinRequest'
      responses:
        '201':
          description: チェックイン成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Checkin'
        '400':
          description: バリデーションエラー
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /checkins/{checkinId}:
    delete:
      summary: チェックイン削除
      tags: [Checkins]
      parameters:
        - name: checkinId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: 削除成功
        '404':
          description: チェックインが見つからない

  # ============================================
  # ユーザー（Users）
  # ============================================
  /users/me:
    get:
      summary: 自分のプロフィール取得
      tags: [Users]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserProfile'

    patch:
      summary: プロフィール更新
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserProfileUpdateRequest'
      responses:
        '200':
          description: 更新成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserProfile'

  # ============================================
  # 進捗（Progress）
  # ============================================
  /users/me/progress:
    get:
      summary: 達成状況取得
      description: 全体の進捗率＋路線別の進捗率
      tags: [Progress]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OverallProgress'
```

---

## APIエンドポイント一覧

| メソッド | パス | 概要 | 認証 |
|----------|------|------|------|
| GET | /stations | 全駅一覧（フィルタ可） | 必要 |
| GET | /stations/search?q= | 駅名検索 | 必要 |
| GET | /stations/:id | 駅詳細 | 必要 |
| GET | /lines | 全路線一覧（進捗付き） | 必要 |
| GET | /lines/:id | 路線詳細（駅一覧付き） | 必要 |
| POST | /checkins | チェックイン登録 | 必要 |
| GET | /checkins | チェックイン履歴 | 必要 |
| DELETE | /checkins/:id | チェックイン削除 | 必要 |
| GET | /users/me | 自分のプロフィール | 必要 |
| PATCH | /users/me | プロフィール更新 | 必要 |
| GET | /users/me/progress | 達成状況 | 必要 |

---

## エラーレスポンス形式

すべてのエラーは統一フォーマットで返す：

```json
{
  "code": "STATION_NOT_FOUND",
  "message": "指定された駅が見つかりません"
}
```

### 主なエラーコード

| HTTPステータス | code | 説明 |
|---------------|------|------|
| 400 | VALIDATION_ERROR | リクエストパラメータ不正 |
| 401 | UNAUTHORIZED | 認証トークンが無効 |
| 403 | FORBIDDEN | 権限なし |
| 404 | STATION_NOT_FOUND | 駅が見つからない |
| 404 | CHECKIN_NOT_FOUND | チェックインが見つからない |
| 409 | DUPLICATE_CHECKIN | 同一駅への重複チェックイン（将来の制限用） |
| 500 | INTERNAL_ERROR | サーバー内部エラー |

---

## Phase 2以降で追加予定のAPI

| メソッド | パス | Phase | 概要 |
|----------|------|-------|------|
| POST | /checkins/:id/photos | Phase 2 | 写真アップロード |
| PUT | /checkins/:id/memo | Phase 2 | メモ更新 |
| POST | /stations/:id/spots | Phase 2 | スポット登録 |
| GET | /badges | Phase 3 | バッジ一覧 |
| GET | /users/me/badges | Phase 3 | 獲得バッジ |
| GET | /stations/random | Phase 3 | ランダム駅提案 |
| POST | /friends | Phase 4 | フレンド追加 |
| GET | /ranking | Phase 4 | ランキング |
