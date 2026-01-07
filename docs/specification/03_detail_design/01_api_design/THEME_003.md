# THEME_003 テンプレートテーマ作成API
- Method: POST
- Path: /api/template-themes
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- themeName（文字列, 必須, 1〜16文字, 空白のみ不可, 例: "日報"）
- ratingName（文字列, 任意, 1〜8文字, 空白のみ不可, 未指定は"重要度", 例: "重要度"）
- questions（配列, 必須, 1〜5件, 例: [{"questionText":"良かった点","defaultAnswer":"","displayOrder":1}]）
- questions[].questionText（文字列, 必須, 1〜50文字, 空白のみ不可, 例: "良かった点"）
- questions[].defaultAnswer（文字列, 任意, 0〜50文字, 例: "学び"）
- questions[].displayOrder（整数, 任意, 1以上, 全件指定または全件未指定, 例: 1）

## Validation Rule
- Rule V001: themeName 必須チェック
  - 対象: themeName
  - 条件: null または 空文字 または 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "テーマ名は必須です。"

- Rule V002: themeName 文字数チェック
  - 対象: themeName
  - 条件: 文字数 > 16
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "テーマ名は16文字以内で入力してください。"

- Rule V003: ratingName 空白チェック
  - 対象: ratingName
  - 条件: 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "評価名は空白のみは使用できません。"

- Rule V004: ratingName 文字数チェック
  - 対象: ratingName
  - 条件: 文字数 > 8
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "評価名は8文字以内で入力してください。"

- Rule V005: questions 必須チェック
  - 対象: questions
  - 条件: null
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "質問リストは必須です。"

- Rule V006: questions 件数チェック
  - 対象: questions
  - 条件: 1件未満 または 5件超
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "質問は1件以上5件以下で入力してください。"

- Rule V007: questionText 必須チェック
  - 対象: questions[].questionText
  - 条件: null または 空文字 または 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "質問文は必須です。"

- Rule V008: questionText 文字数チェック
  - 対象: questions[].questionText
  - 条件: 文字数 > 50
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "質問文は50文字以内で入力してください。"

- Rule V009: defaultAnswer 文字数チェック
  - 対象: questions[].defaultAnswer
  - 条件: 文字数 > 50
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "デフォルト回答は50文字以内で入力してください。"

- Rule V010: 正規化チェック
  - 対象: themeName/ratingName/questions
  - 条件: trim 後の空/上限超過、displayOrder の混在・不連続・0以下
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

- Step 3: 入力値正規化
  - themeName/ratingName/questionText/defaultAnswer を trim
  - ratingName 未指定の場合は "重要度" を設定
  - defaultAnswer 未指定の場合は "" を設定
  - displayOrder の指定有無を確認
    - 全件未指定: リクエスト順に 1..n を付与
    - 全件指定: 1..n 連番のみ許可
  - NG: E-400-VALIDATION を返して終了

- Step 4: テーマ作成
  - user_id + themeName で重複チェック（DBユニーク制約）
  - NG: E-409-TEMPLATE-THEME-DUPLICATE を返して終了
  - トランザクション開始

- Step 5: 質問作成
  - displayOrder 昇順で insert

- Step 6: レスポンス生成
  - TemplateThemeCreateDtoResponse を返却
  - HTTP 201

## Output
### パターン1: 成功
- 型: JSON
- HTTP: 201
- 例:
  {
    "theme": { "id": 1, "themeName": "日報", "ratingName": "重要度" },
    "questions": [
      { "id": 10, "questionText": "良かった点", "defaultAnswer": "", "displayOrder": 1 },
      { "id": 11, "questionText": "改善点", "defaultAnswer": "", "displayOrder": 2 }
    ]
  }

### パターン2: Validation Error（例: V001）
- 型: JSON
- HTTP: 400
- 例:
  {
    "code": "E-400-VALIDATION",
    "message": "テーマ名は必須です。",
    "details": [
      { "field": "themeName", "message": "テーマ名は必須です。" }
    ],
    "operation": "create",
    "themeId": null
  }

### パターン3: 業務エラー（重複）
- 型: JSON
- HTTP: 409
- 例:
  {
    "code": "E-409-TEMPLATE-THEME-DUPLICATE",
    "message": "同じテーマ名が既に存在します。",
    "details": null,
    "operation": "create",
    "themeId": null
  }

### パターン4: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "create",
    "themeId": null
  }

### パターン5: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "create",
    "themeId": null
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: 入力バリデーションNGまたは正規化失敗
  - Input例: themeName = ""
  - 検出Step: Step 2 / Step 3
  - Output例: Output パターン2

- E-409-TEMPLATE-THEME-DUPLICATE
  - 発生条件: user_id + themeName の一意制約違反
  - Input例: themeName = "日報"（既存）
  - 検出Step: Step 4
  - Output例: Output パターン3

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン4

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 4/Step 5
  - Output例: Output パターン5

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "create",
      "themeId": null
    }
