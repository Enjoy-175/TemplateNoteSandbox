# NOTE_002 メモ管理一覧API
- Method: GET
- Path: /api/notes:management
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- pageSize（整数, 任意, 1以上, 未指定は7, pageToken指定時はtokenのsizeと一致必須, 例: 7）
- pageToken（文字列, 任意, Base64URL("page={n}&size={m}"), 例: "cGFnZT0yJnNpemU9Nw"）
- title（文字列, 任意, trim後1文字以上, 部分一致, 例: "振り返り"）
- categoryId（整数, 任意, 正の整数, categoryIds 指定時は無視, 例: 3）
- categoryIds（文字列, 任意, 正の整数のCSV・重複不可, 例: "1,3,5"）
- themeId（整数, 任意, 正の整数, themeIds 指定時は無視, 例: 2）
- themeIds（文字列, 任意, 正の整数のCSV・重複不可, 例: "2,4"）
- tagIds（文字列, 任意, 正の整数のCSV・重複不可, 例: "5,6"）
- eventDateFrom（日付文字列, 任意, YYYY-MM-DD, 例: "2025-12-01"）
- eventDateTo（日付文字列, 任意, YYYY-MM-DD, 例: "2025-12-31"）
- ratingScoreMin（整数, 任意, 0〜5, 例: 1）
- ratingScoreMax（整数, 任意, 0〜5, 例: 5）
- displayPriority（文字列, 任意, low/normal/priority のCSV・重複不可, 例: "normal,priority"）
- orderBys（文字列, 任意, eventDate|ratingScore|title と asc|desc のCSV, 未指定は eventDate:desc, 例: "eventDate:desc"）

## Validation Rule
- Rule V001: pageSize 正数チェック
  - 対象: pageSize
  - 条件: pageSize <= 0
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V002: pageToken 形式チェック
  - 対象: pageToken
  - 条件: Base64URL で復号できない、または "page"/"size" が欠落/不正
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V003: pageSize/pageToken 整合性チェック
  - 対象: pageSize, pageToken
  - 条件: pageToken の size と pageSize が一致しない
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V004: categoryId/categoryIds 形式チェック
  - 対象: categoryId, categoryIds
  - 条件: 正の整数以外、CSV内の空要素、重複
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V005: themeId/themeIds 形式チェック
  - 対象: themeId, themeIds
  - 条件: 正の整数以外、CSV内の空要素、重複
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V006: tagIds 形式チェック
  - 対象: tagIds
  - 条件: 正の整数以外、CSV内の空要素、重複
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V007: 日付範囲チェック
  - 対象: eventDateFrom, eventDateTo
  - 条件: YYYY-MM-DD 形式でない、または from > to
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V008: 評価範囲チェック
  - 対象: ratingScoreMin, ratingScoreMax
  - 条件: 0〜5以外、または min > max
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V009: displayPriority 形式チェック
  - 対象: displayPriority
  - 条件: low/normal/priority 以外、重複
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V010: orderBys 形式チェック
  - 対象: orderBys
  - 条件: key=eventDate|ratingScore|title 以外、order=asc|desc 以外、キー重複
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

### チェック順序
1. V001 → NGなら終了
2. V002 → NGなら終了
3. V003 → NGなら終了
4. V004 → NGなら終了
5. V005 → NGなら終了
6. V006 → NGなら終了
7. V007 → NGなら終了
8. V008 → NGなら終了
9. V009 → NGなら終了
10. V010 → NGなら終了
11. 全てOKなら次の処理へ

## Process
- Step 1: セッション認証
  - SessionUserInterceptor でセッション確認
  - NG: E-401-UNAUTHORIZED を返して終了

- Step 2: 入力バリデーション
  - Validation Rule を順に適用
  - NG: E-400-VALIDATION を返して終了

- Step 3: ページ条件確定
  - pageToken があれば page/size を復元
  - pageToken なしの場合は page=1, size=pageSize(未指定は7)

- Step 4: メモ一覧取得
  - NOTE_001 と同じ検索条件でメモを取得

- Step 5: 付帯データ取得
  - categories/tags/templateThemes を全件取得

- Step 6: 次ページトークン生成
  - 検索結果が残っている場合のみ nextPageToken を設定

- Step 7: レスポンス生成
  - notes/searchResultCount/nextPageToken/categories/tags/templateThemes を返却

## Output
### パターン1: 成功
- 型: JSON
- HTTP: 200
- 例:
  {
    "notes": [
      {
        "id": 123,
        "themeId": 1,
        "categoryId": 3,
        "title": "振り返り",
        "eventDate": "2025-12-27",
        "ratingScore": 4,
        "displayPriority": "normal",
        "tagIds": [5, 6]
      }
    ],
    "searchResultCount": 1,
    "nextPageToken": "cGFnZT0yJnNpemU9Nw",
    "categories": [
      { "id": 1, "catName": "業務", "description": "仕事に関するメモ" }
    ],
    "tags": [
      { "id": 1, "tagKey": "type", "tagValue": "memo" }
    ],
    "templateThemes": [
      { "id": 1, "themeName": "週次レビュー", "ratingName": "満足度" }
    ]
  }

### パターン2: Validation Error（例: V002）
- 型: JSON
- HTTP: 400
- 例:
  {
    "code": "E-400-VALIDATION",
    "message": "入力値が不正です。",
    "details": [
      { "field": "pageToken", "message": "入力値が不正です。" }
    ],
    "operation": "fetch",
    "noteId": null
  }

### パターン3: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "fetch",
    "noteId": null
  }

### パターン4: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "fetch",
    "noteId": null
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: pageToken/検索条件/並び順の入力不正
  - Input例: pageToken = "invalid"
  - 検出Step: Step 2
  - Output例: Output パターン2

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン3

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 4 または Step 5
  - Output例: Output パターン4

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "fetch",
      "noteId": null
    }
