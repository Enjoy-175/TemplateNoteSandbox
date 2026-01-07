# THEME_002 テンプレートテーマ詳細取得API
- Method: GET
- Path: /api/template-themes/{id}
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- id（整数, 必須, 正の整数, 例: 1）

## Validation Rule
- Rule V001: id 必須チェック
  - 対象: id
  - 条件: null
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V002: id 正数チェック
  - 対象: id
  - 条件: id <= 0
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

### チェック順序
1. V001 → NGなら終了
2. V002 → NGなら終了
3. 全てOKなら次の処理へ

## Process
- Step 1: セッション認証
  - SessionUserInterceptor でセッション確認
  - NG: E-401-UNAUTHORIZED を返して終了

- Step 2: 入力バリデーション
  - Validation Rule を順に適用
  - NG: E-400-VALIDATION を返して終了

- Step 3: テーマ取得
  - id でテーマを取得
  - 条件: テーマが存在する
  - NG: E-404-TEMPLATE-THEME-NOT-FOUND を返して終了

- Step 4: 所有者チェック
  - 条件: 取得したテーマの user_id == sessionUserId
  - NG: E-403-TEMPLATE-THEME-FORBIDDEN を返して終了

- Step 5: 質問一覧取得
  - user_id + theme_id で質問を取得
  - ORDER BY display_order ASC

- Step 6: レスポンス生成
  - TemplateThemeDetailDtoResponse を返却

## Output
### パターン1: 成功
- 型: JSON
- HTTP: 200
- 例:
  {
    "theme": { "id": 1, "themeName": "日報", "ratingName": "重要度" },
    "questions": [
      { "id": 10, "questionText": "良かった点", "defaultAnswer": "", "displayOrder": 1 },
      { "id": 11, "questionText": "改善点", "defaultAnswer": "", "displayOrder": 2 }
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
      { "field": "id", "message": "入力値が不正です。" }
    ],
    "operation": "fetch",
    "themeId": 1
  }

### パターン3: 業務エラー（存在しない）
- 型: JSON
- HTTP: 404
- 例:
  {
    "code": "E-404-TEMPLATE-THEME-NOT-FOUND",
    "message": "テーマが存在しません。",
    "details": null,
    "operation": "fetch",
    "themeId": 1
  }

### パターン4: 業務エラー（権限なし）
- 型: JSON
- HTTP: 403
- 例:
  {
    "code": "E-403-TEMPLATE-THEME-FORBIDDEN",
    "message": "他のユーザーのテーマは操作できません。",
    "details": null,
    "operation": "fetch",
    "themeId": 1
  }

### パターン5: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "fetch",
    "themeId": 1
  }

### パターン6: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "fetch",
    "themeId": 1
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: id の必須/形式チェックNG
  - Input例: id = 0
  - 検出Step: Step 2
  - Output例: Output パターン2

- E-404-TEMPLATE-THEME-NOT-FOUND
  - 発生条件: id に一致するテーマが存在しない
  - Input例: id = 9999
  - 検出Step: Step 3
  - Output例: Output パターン3

- E-403-TEMPLATE-THEME-FORBIDDEN
  - 発生条件: テーマの user_id が sessionUserId と一致しない
  - Input例: id = 1（他ユーザーのテーマ）
  - 検出Step: Step 4
  - Output例: Output パターン4

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン5

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 3/Step 5
  - Output例: Output パターン6

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "fetch",
      "themeId": 1
    }
