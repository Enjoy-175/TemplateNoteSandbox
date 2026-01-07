# NOTE_005 メモ作成API
- Method: POST
- Path: /api/notes
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- themeId（整数, 必須, 正の整数, 例: 1）
- title（文字列, 必須, 1〜50文字, 空白のみ不可, 例: "振り返り"）
- eventDate（日付, 必須, YYYY-MM-DD, 例: "2025-12-27"）
- categoryId（整数, 任意, 正の整数, 例: 3）
- ratingScore（整数, 任意, 0〜5, 未指定は0, 例: 4）
- displayPriority（文字列, 任意, low/normal/priority, 未指定はnormal, 例: "normal"）
- answers（配列, 任意, 例: [{"questionId":10,"answer":"良かった点","referenceUrl":"https://example.com"}]）
- tagIds（配列[整数], 任意, 0〜3件, 重複不可, 例: [5,6]）

## Validation Rule
- Rule V001: themeId 必須チェック
  - 対象: themeId
  - 条件: null
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "テーマIDは必須です。"

- Rule V002: title 必須/形式チェック
  - 対象: title
  - 条件: null または 空文字 または 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "タイトルは必須です。"

- Rule V003: title 文字数チェック
  - 対象: title
  - 条件: 文字数 > 50
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "タイトルは50文字以内で入力してください。"

- Rule V004: eventDate 必須チェック
  - 対象: eventDate
  - 条件: null
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "記録日は必須です。"

- Rule V005: ratingScore 範囲チェック
  - 対象: ratingScore
  - 条件: 0〜5 以外
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "評価は0〜5で入力してください。"

- Rule V006: displayPriority 形式チェック
  - 対象: displayPriority
  - 条件: low/normal/priority 以外
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "表示優先度は low/normal/priority のいずれかで入力してください。"

- Rule V007: answers 要素チェック
  - 対象: answers[*]
  - 条件: questionId が null、answer が null、answer > 80文字
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "入力値が不正です。"

- Rule V008: tagIds 件数/重複チェック
  - 対象: tagIds
  - 条件: 4件以上、重複、null を含む
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "タグは最大3件までです。"

- Rule V009: 正規化チェック
  - 対象: title/displayPriority/answers/tagIds
  - 条件: trim 後の空/上限超過/重複
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
10. 全てOKなら次の処理へ

## Process
- Step 1: セッション認証
  - SessionUserInterceptor でセッション確認
  - NG: E-401-UNAUTHORIZED を返して終了

- Step 2: 入力バリデーション
  - Validation Rule を順に適用
  - NG: E-400-VALIDATION を返して終了

- Step 3: 入力正規化
  - title を trim
  - ratingScore 未指定は 0
  - displayPriority 未指定は normal
  - answers は questionId 重複不可、answer/URL を正規化
  - tagIds は重複除外（重複はNG）

- Step 4: 参照整合性チェック
  - themeId が存在し、所有者が sessionUserId であること
  - categoryId 指定時は存在/所有者を確認
  - tagIds 指定時は全て存在/所有者を確認
  - NG: 対応する 403/404 エラー

- Step 5: 質問/回答整合性チェック
  - themeId 配下の active 質問を取得
  - answers の questionId が全て存在すること
  - NG: E-400-VALIDATION

- Step 6: メモ作成
  - notes を insert
  - note_answers を質問数分作成（未指定は空文字で補完）
  - tagIds 指定時は note_tags を作成
  - トランザクション開始
  - 失敗: ロールバックし E-500-DB

- Step 7: レスポンス生成
  - 作成したメモ（answers/tagIds含む）を返却
  - Location ヘッダに /api/notes/{id} を設定

## Output
### パターン1: 成功
- 型: JSON
- HTTP: 201
- 例:
  {
    "id": 123,
    "themeId": 1,
    "categoryId": 3,
    "title": "振り返り",
    "eventDate": "2025-12-27",
    "ratingScore": 4,
    "displayPriority": "normal",
    "answers": [
      { "questionId": 10, "answer": "良かった点", "referenceUrl": "https://example.com/ref-1" },
      { "questionId": 11, "answer": "", "referenceUrl": "" }
    ],
    "tagIds": [5, 6]
  }

### パターン2: Validation Error（例: V002）
- 型: JSON
- HTTP: 400
- 例:
  {
    "code": "E-400-VALIDATION",
    "message": "タイトルは必須です。",
    "details": [
      { "field": "title", "message": "タイトルは必須です。" }
    ],
    "operation": "create",
    "noteId": null
  }

### パターン3: 業務エラー（テーマ不正）
- 型: JSON
- HTTP: 404
- 例:
  {
    "code": "E-404-TEMPLATE-THEME-NOT-FOUND",
    "message": "テーマが存在しません。",
    "details": null,
    "operation": "create",
    "noteId": null
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
    "noteId": null
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
    "noteId": null
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: 入力バリデーションNG、正規化NG、質問IDの不一致
  - Input例: title = ""
  - 検出Step: Step 2 / Step 3 / Step 5
  - Output例: Output パターン2

- E-404-TEMPLATE-THEME-NOT-FOUND
  - 発生条件: themeId が存在しない
  - Input例: themeId = 999
  - 検出Step: Step 4
  - Output例: Output パターン3

- E-403-TEMPLATE-THEME-FORBIDDEN
  - 発生条件: 他ユーザーのテーマを指定
  - Input例: themeId = 2（他ユーザー所有）
  - 検出Step: Step 4
  - Output例:
    {
      "code": "E-403-TEMPLATE-THEME-FORBIDDEN",
      "message": "他のユーザーのテーマは操作できません。",
      "details": null,
      "operation": "create",
      "noteId": null
    }

- E-404-CATEGORY-NOT-FOUND
  - 発生条件: categoryId が存在しない
  - 検出Step: Step 4
  - Output例:
    {
      "code": "E-404-CATEGORY-NOT-FOUND",
      "message": "カテゴリが存在しません。",
      "details": null,
      "operation": "create",
      "noteId": null
    }

- E-403-CATEGORY-FORBIDDEN
  - 発生条件: 他ユーザーのカテゴリを指定
  - 検出Step: Step 4
  - Output例:
    {
      "code": "E-403-CATEGORY-FORBIDDEN",
      "message": "他のユーザーのカテゴリは操作できません。",
      "details": null,
      "operation": "create",
      "noteId": null
    }

- E-404-TAG-NOT-FOUND
  - 発生条件: tagIds に存在しないタグが含まれる
  - 検出Step: Step 4
  - Output例:
    {
      "code": "E-404-TAG-NOT-FOUND",
      "message": "タグが存在しません。",
      "details": null,
      "operation": "create",
      "noteId": null
    }

- E-403-TAG-FORBIDDEN
  - 発生条件: tagIds に他ユーザーのタグが含まれる
  - 検出Step: Step 4
  - Output例:
    {
      "code": "E-403-TAG-FORBIDDEN",
      "message": "他のユーザーのタグは操作できません。",
      "details": null,
      "operation": "create",
      "noteId": null
    }

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン4

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 6
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
      "noteId": null
    }
