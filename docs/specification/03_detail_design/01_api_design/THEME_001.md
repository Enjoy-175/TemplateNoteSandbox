# THEME_001 テンプレートテーマ一覧取得API
- Method: GET
- Path: /api/template-themes
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- includeQuestions（文字列, 任意, "true" 指定時のみ質問を含める, 例: "true"）

## Validation Rule
- なし（includeQuestions の値は検証せず、"true" の場合のみ質問を含める）

## Process
- Step 1: セッション認証
  - SessionUserInterceptor でセッション確認
  - NG: E-401-UNAUTHORIZED を返して終了

- Step 2: includeQuestions 判定
  - 条件: includeQuestions == "true"
  - true: 質問を含める
  - false/未指定: 質問を含めない

- Step 3: テーマ一覧取得
  - user_id で絞り込み
  - ORDER BY id ASC

- Step 4: 質問一覧取得（includeQuestions=true の場合のみ）
  - 対象テーマIDで質問を取得
  - ORDER BY theme_id ASC, display_order ASC

- Step 5: レスポンス生成
  - includeQuestions=false: TemplateThemeDtoResponse の配列を返却
  - includeQuestions=true: TemplateThemeWithQuestionsDtoResponse の配列を返却

## Output
### パターン1: 成功（includeQuestions 未指定/false）
- 型: JSON（配列）
- HTTP: 200
- 例:
  [
    { "id": 1, "themeName": "日報", "ratingName": "重要度" },
    { "id": 2, "themeName": "週次振り返り", "ratingName": "重要度" }
  ]

### パターン2: 成功（includeQuestions=true）
- 型: JSON（配列）
- HTTP: 200
- 例:
  [
    {
      "id": 1,
      "themeName": "日報",
      "ratingName": "重要度",
      "questions": [
        { "id": 10, "questionText": "良かった点", "defaultAnswer": "", "displayOrder": 1 },
        { "id": 11, "questionText": "改善点", "defaultAnswer": "", "displayOrder": 2 }
      ]
    }
  ]

### パターン3: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "fetch",
    "themeId": null
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
    "themeId": null
  }

## Error 定義
- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン3

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 3/Step 4
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
      "themeId": null
    }
