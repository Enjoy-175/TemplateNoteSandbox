# NOTE_007 メモ削除API
- Method: DELETE
- Path: /api/notes/{id}
- 認証: 要（セッション必須）
- 備考: 冪等（存在しない/他ユーザーのメモでも 204）

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

- Step 3: 削除
  - notes を user_id + id で削除
  - 対象が存在しない場合も成功扱い
  - トランザクション開始
  - 失敗: ロールバックし E-500-DB

- Step 4: レスポンス生成
  - No Content を返却

## Output
### パターン1: 成功
- 型: なし
- HTTP: 204

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
    "operation": "delete",
    "noteId": 123
  }

### パターン3: 認証エラー
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "delete",
    "noteId": 123
  }

### パターン4: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "delete",
    "noteId": 123
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: id の必須/形式チェックNG
  - Input例: id = 0
  - 検出Step: Step 2
  - Output例: Output パターン2

- E-401-UNAUTHORIZED
  - 発生条件: セッション未確立または期限切れ
  - 検出Step: Step 1
  - Output例: Output パターン3

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 3
  - Output例: Output パターン4

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "delete",
      "noteId": 123
    }
