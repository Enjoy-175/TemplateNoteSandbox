# NOTE_004 メモMarkdown取得API
- Method: GET
- Path: /api/notes/{id}/markdown
- 認証: 要（セッション必須）

## Input
- sessionUserId（UUID, 必須, セッションから取得, 例: "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b"）
- id（整数, 必須, 正の整数, 例: 123）

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

- Step 3: メモ詳細取得
  - id + sessionUserId でメモ詳細を取得
  - 条件: 該当メモが存在する
  - NG: E-404-NOTE-NOT-FOUND を返して終了

- Step 4: 参照データ取得
  - テーマ/カテゴリ/タグ/質問を取得（存在しない場合は空扱い）

- Step 5: Markdown 生成
  - 回答は display_order 昇順で出力
  - ファイル名はタイトルをサニタイズして生成（空タイトルは note-{id}）

- Step 6: レスポンス生成
  - Content-Type: text/markdown; charset=UTF-8
  - Content-Disposition: attachment; filename="{asciiFileName}"; filename*=UTF-8''{utf8FileName}

## Output
### パターン1: 成功
- 型: bytes（text/markdown）
- HTTP: 200
- 例（本文抜粋）:
  # 振り返り

  ---

  ## メタ情報
  - **テンプレート**: 週次レビュー
  - **イベント日**: 2025-12-27

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
    "noteId": 123
  }

### パターン3: 業務エラー（存在しない）
- 型: JSON
- HTTP: 404
- 例:
  {
    "code": "E-404-NOTE-NOT-FOUND",
    "message": "メモが存在しません。",
    "details": null,
    "operation": "fetch",
    "noteId": 123
  }

### パターン4: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "fetch",
    "noteId": 123
  }

### パターン5: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "fetch",
    "noteId": 123
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: id の必須/形式チェックNG
  - Input例: id = 0
  - 検出Step: Step 2
  - Output例: Output パターン2

- E-404-NOTE-NOT-FOUND
  - 発生条件: id に一致するメモが存在しない（他ユーザーのメモ含む）
  - Input例: id = 9999
  - 検出Step: Step 3
  - Output例: Output パターン3

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン4

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 3 または Step 4
  - Output例: Output パターン5

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "fetch",
      "noteId": 123
    }
