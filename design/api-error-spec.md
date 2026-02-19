# エラー仕様書（フェーズ1 / MVP）

本資料は、部活動向け業務支援システムにおける\
**エラー仕様（共通仕様およびAPI別異常系定義）** を定義する。

------------------------------------------------------------------------

## 1. 共通エラーレスポンス形式

### 1.1 ErrorResponse

``` json
{
  "code": "VALIDATION_ERROR",
  "message": "入力内容に誤りがあります",
  "details": [
    { "field": "targetDate", "reason": "必須です" }
  ],
  "traceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

### 項目説明

  項目      説明
  --------- ------------------------------------
  code      アプリケーションエラーコード
  message   利用者向け簡易メッセージ
  details   フィールド単位のエラー詳細（任意）
  traceId   ログ追跡用ID

------------------------------------------------------------------------

## 2. HTTPステータス一覧

  HTTP   意味
  ------ ---------------------------------
  400    Bad Request（入力不備）
  401    Unauthorized（未認証）
  403    Forbidden（権限不足）
  404    Not Found（対象なし）
  409    Conflict（状態不整合・排他）
  500    Internal Server Error（想定外）

------------------------------------------------------------------------

## 3. エラーコード一覧

  code                   HTTP   説明
  ---------------------- ------ --------------------
  VALIDATION_ERROR       400    入力不備・形式不正
  UNAUTHORIZED           401    未認証
  FORBIDDEN              403    権限不足
  NOT_FOUND              404    対象が存在しない
  STATE_CONFLICT         409    不正な状態遷移
  CONCURRENCY_CONFLICT   409    排他制御エラー
  INTERNAL_ERROR         500    想定外エラー

------------------------------------------------------------------------

## 4. 欠席申請API 異常系

### 4.1 作成・編集

  HTTP   code               条件
  ------ ------------------ --------------
  400    VALIDATION_ERROR   必須項目不足
  401    UNAUTHORIZED       未認証
  403    FORBIDDEN          STUDENT以外
  409    STATE_CONFLICT     編集不可状態
  500    INTERNAL_ERROR     想定外

### 4.2 submit / cancel

  HTTP   code             条件
  ------ ---------------- --------------
  403    FORBIDDEN        他人の申請
  404    NOT_FOUND        ID不存在
  409    STATE_CONFLICT   不正状態遷移

### 4.3 approve / return / reject

  HTTP   code             条件
  ------ ---------------- ---------------
  403    FORBIDDEN        COACH以外
  404    NOT_FOUND        ID不存在
  409    STATE_CONFLICT   SUBMITTED以外

------------------------------------------------------------------------

## 5. 試合結果API 異常系

### 5.1 作成・編集

  HTTP   code               条件
  ------ ------------------ --------------
  400    VALIDATION_ERROR   必須項目不足
  403    FORBIDDEN          STUDENT以外
  409    STATE_CONFLICT     編集不可状態

### 5.2 submit / cancel / approve / return / reject

  HTTP   code             条件
  ------ ---------------- --------------
  403    FORBIDDEN        権限不足
  404    NOT_FOUND        ID不存在
  409    STATE_CONFLICT   不正状態遷移

------------------------------------------------------------------------

## 6. 練習スケジュールAPI 異常系

  HTTP   code               条件
  ------ ------------------ -----------
  400    VALIDATION_ERROR   必須不足
  403    FORBIDDEN          COACH以外
  404    NOT_FOUND          ID不存在

------------------------------------------------------------------------

## 7. 通知API 異常系

  HTTP   code        条件
  ------ ----------- ----------------
  403    FORBIDDEN   他人の通知操作
  404    NOT_FOUND   ID不存在

------------------------------------------------------------------------

## 8. 管理API 異常系

  HTTP   code               条件
  ------ ------------------ -----------
  403    FORBIDDEN          ADMIN以外
  400    VALIDATION_ERROR   入力不備
  404    NOT_FOUND          ID不存在

------------------------------------------------------------------------

## 9. 状態遷移制約（共通）

  現在状態           実行操作   結果
  ------------------ ---------- -----------
  DRAFT / RETURNED   submit     SUBMITTED
  SUBMITTED          approve    APPROVED
  SUBMITTED          return     RETURNED
  SUBMITTED          reject     REJECTED
  SUBMITTED          cancel     CANCELLED

※ 不正な遷移は 409 STATE_CONFLICT を返却する。

------------------------------------------------------------------------

## 10. 実装指針

-   業務エラーは明示的に例外クラスで管理する
-   ControllerAdviceで共通ハンドリングする
-   traceIdはログと相関させる
-   状態遷移チェックはService層で実施する
