# API設計書（フェーズ1 / MVP）

（Request / Response：成功時のみ）

本資料は、部活動向け業務支援システムにおけるAPI仕様を定義する。\
※本書は「URIは確定」「成功時のRequest/Response例を全APIに付与」した版である。

------------------------------------------------------------------------

## 1. 共通仕様

### ベースURL

`https://{host}/api/v1`

### 認証方式

-   JWT（Bearerトークン）
-   Header: `Authorization: Bearer {token}`

### ロール

-   STUDENT
-   COACH
-   ADMIN

### 共通：一覧レスポンス形式

``` json
{
  "items": [],
  "page": 1,
  "size": 20,
  "total": 0
}
```

------------------------------------------------------------------------

## 2. 欠席申請API

### POST /absence-requests

欠席申請を作成（下書き or 提出）

#### Request

``` json
{
  "targetDate": "2026-02-19",
  "type": "ABSENCE",
  "reason": "体調不良のため",
  "submit": false
}
```

#### Response 201

``` json
{
  "id": "ar_001",
  "targetDate": "2026-02-19",
  "type": "ABSENCE",
  "reason": "体調不良のため",
  "status": "DRAFT",
  "createdAt": "2026-02-19T10:00:00+09:00",
  "updatedAt": "2026-02-19T10:00:00+09:00"
}
```

------------------------------------------------------------------------

### GET /absence-requests

欠席申請一覧取得

#### Response 200

``` json
{
  "items": [
    {
      "id": "ar_001",
      "targetDate": "2026-02-19",
      "type": "ABSENCE",
      "status": "SUBMITTED",
      "updatedAt": "2026-02-19T10:05:00+09:00"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

------------------------------------------------------------------------

### GET /absence-requests/{id}

欠席申請詳細取得（履歴含む）

#### Response 200

``` json
{
  "id": "ar_001",
  "targetDate": "2026-02-19",
  "type": "ABSENCE",
  "reason": "体調不良のため",
  "status": "SUBMITTED",
  "history": [
    {
      "action": "SUBMIT",
      "actor": { "id": "u_001", "name": "佐藤 薫", "role": "STUDENT" },
      "comment": null,
      "actedAt": "2026-02-19T10:05:00+09:00"
    }
  ],
  "createdAt": "2026-02-19T10:00:00+09:00",
  "updatedAt": "2026-02-19T10:05:00+09:00"
}
```

------------------------------------------------------------------------

### PATCH /absence-requests/{id}

欠席申請編集（DRAFT / RETURNEDのみ）

#### Request

``` json
{
  "targetDate": "2026-02-20",
  "type": "LATE",
  "reason": "通院のため"
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "DRAFT",
  "updatedAt": "2026-02-19T10:10:00+09:00"
}
```

------------------------------------------------------------------------

### POST /absence-requests/{id}/submit

提出

#### Request（任意）

``` json
{
  "comment": null
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "SUBMITTED"
}
```

------------------------------------------------------------------------

### POST /absence-requests/{id}/cancel

取消

#### Request（任意）

``` json
{
  "comment": "都合がついたため"
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "CANCELLED"
}
```

------------------------------------------------------------------------

### POST /absence-requests/{id}/approve

承認（COACHのみ）

#### Request（任意）

``` json
{
  "comment": "了解しました"
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "APPROVED"
}
```

------------------------------------------------------------------------

### POST /absence-requests/{id}/return

差し戻し（COACHのみ）

#### Request

``` json
{
  "comment": "理由をもう少し具体的にしてください"
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "RETURNED"
}
```

------------------------------------------------------------------------

### POST /absence-requests/{id}/reject

却下（COACHのみ）

#### Request

``` json
{
  "comment": "提出期限を過ぎています"
}
```

#### Response 200

``` json
{
  "id": "ar_001",
  "status": "REJECTED"
}
```

------------------------------------------------------------------------

## 3. 試合結果API

### POST /match-reports

作成

#### Request

``` json
{
  "matchDate": "2026-02-18",
  "opponent": "○○高校",
  "score": "6-4, 3-6, 10-8",
  "result": "WIN",
  "comment": "接戦でした",
  "submit": false
}
```

#### Response 201

``` json
{
  "id": "mr_001",
  "matchDate": "2026-02-18",
  "opponent": "○○高校",
  "score": "6-4, 3-6, 10-8",
  "result": "WIN",
  "comment": "接戦でした",
  "status": "DRAFT",
  "isOfficial": false,
  "createdAt": "2026-02-19T10:00:00+09:00",
  "updatedAt": "2026-02-19T10:00:00+09:00"
}
```

------------------------------------------------------------------------

### GET /match-reports

一覧

#### Response 200

``` json
{
  "items": [
    {
      "id": "mr_001",
      "matchDate": "2026-02-18",
      "opponent": "○○高校",
      "result": "WIN",
      "status": "SUBMITTED",
      "isOfficial": false,
      "updatedAt": "2026-02-19T10:00:00+09:00"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

------------------------------------------------------------------------

### GET /match-reports/{id}

詳細（履歴含む）

#### Response 200

``` json
{
  "id": "mr_001",
  "matchDate": "2026-02-18",
  "opponent": "○○高校",
  "score": "6-4, 3-6, 10-8",
  "result": "WIN",
  "comment": "接戦でした",
  "status": "SUBMITTED",
  "isOfficial": false,
  "history": [
    {
      "action": "SUBMIT",
      "actor": { "id": "u_001", "name": "佐藤 薫", "role": "STUDENT" },
      "comment": null,
      "actedAt": "2026-02-19T10:00:00+09:00"
    }
  ],
  "createdAt": "2026-02-19T10:00:00+09:00",
  "updatedAt": "2026-02-19T10:00:00+09:00"
}
```

------------------------------------------------------------------------

### PATCH /match-reports/{id}

編集（DRAFT / RETURNEDのみ）

#### Request

``` json
{
  "score": "6-4, 6-3",
  "comment": "内容を修正しました"
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "DRAFT",
  "updatedAt": "2026-02-19T10:10:00+09:00"
}
```

------------------------------------------------------------------------

### POST /match-reports/{id}/submit

提出

#### Request（任意）

``` json
{
  "comment": null
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "SUBMITTED"
}
```

------------------------------------------------------------------------

### POST /match-reports/{id}/cancel

取消

#### Request（任意）

``` json
{
  "comment": "誤入力のため"
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "CANCELLED"
}
```

------------------------------------------------------------------------

### POST /match-reports/{id}/approve

承認（isOfficial=true）

#### Request（任意）

``` json
{
  "comment": "公式記録として承認します"
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "APPROVED",
  "isOfficial": true
}
```

------------------------------------------------------------------------

### POST /match-reports/{id}/return

差し戻し

#### Request

``` json
{
  "comment": "スコアを再確認してください"
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "RETURNED"
}
```

------------------------------------------------------------------------

### POST /match-reports/{id}/reject

却下

#### Request

``` json
{
  "comment": "内容不備のため却下します"
}
```

#### Response 200

``` json
{
  "id": "mr_001",
  "status": "REJECTED"
}
```

------------------------------------------------------------------------

## 4. 練習スケジュールAPI

### POST /schedules

登録（COACHのみ）

#### Request

``` json
{
  "date": "2026-02-20",
  "startTime": "18:00",
  "endTime": "20:00",
  "location": "第1コート",
  "menu": "基礎練",
  "note": "雨天中止"
}
```

#### Response 201

``` json
{
  "id": "sc_001",
  "date": "2026-02-20",
  "startTime": "18:00",
  "endTime": "20:00",
  "location": "第1コート",
  "menu": "基礎練",
  "note": "雨天中止"
}
```

------------------------------------------------------------------------

### GET /schedules

一覧

#### Response 200

``` json
{
  "items": [
    {
      "id": "sc_001",
      "date": "2026-02-20",
      "startTime": "18:00",
      "endTime": "20:00",
      "location": "第1コート",
      "menu": "基礎練"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

------------------------------------------------------------------------

### PATCH /schedules/{id}

編集（COACHのみ）

#### Request

``` json
{
  "location": "第2コート",
  "note": "コート変更"
}
```

#### Response 200

``` json
{
  "id": "sc_001",
  "date": "2026-02-20",
  "startTime": "18:00",
  "endTime": "20:00",
  "location": "第2コート",
  "menu": "基礎練",
  "note": "コート変更"
}
```

------------------------------------------------------------------------

### DELETE /schedules/{id}

削除（COACHのみ）

#### Response 204

(No Content)

------------------------------------------------------------------------

## 5. 通知API

### GET /notifications

通知一覧取得

#### Response 200

``` json
{
  "items": [
    {
      "id": "nt_001",
      "type": "ABSENCE_APPROVED",
      "targetCategory": "ABSENCE",
      "targetId": "ar_001",
      "isRead": false,
      "createdAt": "2026-02-19T11:00:00+09:00"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

------------------------------------------------------------------------

### POST /notifications/{id}/read

既読化

#### Response 200

``` json
{
  "id": "nt_001",
  "isRead": true
}
```

------------------------------------------------------------------------

### POST /notifications/read-all

全件既読

#### Response 204

(No Content)

------------------------------------------------------------------------

## 6. 管理API（ADMIN）

### /admin/clubs

部活管理

#### GET /admin/clubs（一覧） Response 200

``` json
{
  "items": [
    { "id": "c_001", "name": "テニス部" }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

#### POST /admin/clubs（作成） Request

``` json
{
  "name": "テニス部"
}
```

#### POST /admin/clubs Response 201

``` json
{
  "id": "c_001",
  "name": "テニス部"
}
```

#### PATCH /admin/clubs/{id}（更新） Request

``` json
{
  "name": "テニス部（新）"
}
```

#### PATCH /admin/clubs/{id} Response 200

``` json
{
  "id": "c_001",
  "name": "テニス部（新）"
}
```

#### DELETE /admin/clubs/{id}（削除） Response 204

(No Content)

------------------------------------------------------------------------

### /admin/users

ユーザー管理

#### GET /admin/users（一覧） Response 200

``` json
{
  "items": [
    { "id": "u_001", "userId": "sato", "name": "佐藤 薫", "role": "STUDENT", "clubId": "c_001" }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

#### POST /admin/users（作成） Request

``` json
{
  "userId": "sato",
  "name": "佐藤 薫",
  "role": "STUDENT",
  "clubId": "c_001",
  "password": "plaintext"
}
```

#### POST /admin/users Response 201

``` json
{
  "id": "u_001",
  "userId": "sato",
  "name": "佐藤 薫",
  "role": "STUDENT",
  "clubId": "c_001"
}
```

#### PATCH /admin/users/{id}（更新） Request

``` json
{
  "name": "佐藤 薫（更新）",
  "role": "COACH"
}
```

#### PATCH /admin/users/{id} Response 200

``` json
{
  "id": "u_001",
  "userId": "sato",
  "name": "佐藤 薫（更新）",
  "role": "COACH",
  "clubId": "c_001"
}
```

------------------------------------------------------------------------

### /admin/audit-logs

監査ログ参照

#### GET /admin/audit-logs Response 200

``` json
{
  "items": [
    {
      "id": "al_001",
      "action": "ABSENCE_SUBMIT",
      "actorUserId": "u_001",
      "targetCategory": "ABSENCE",
      "targetId": "ar_001",
      "comment": null,
      "actedAt": "2026-02-19T10:05:00+09:00"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```
