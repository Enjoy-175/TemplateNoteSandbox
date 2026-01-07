# AUTH_002 ログアウトAPI
- Method: DELETE
- Path: /api/auth/logout
- 認証: 不要（セッションなしでも 204）

## Input
- なし

## Validation Rule
- なし（入力項目なし）

### チェック順序
- なし

## Process
- Step 1: セッション無効化
  - request からセッションを取得し、存在する場合のみ invalidate を実行
  - セッションが存在しない場合はそのまま継続

- Step 2: レスポンス生成
  - No Content を返却

## Output
### パターン1: 成功
- 型: なし
- HTTP: 204

### パターン2: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-UNEXPECTED",
    "message": "予期しないエラーが発生しました。",
    "details": null,
    "operation": "delete"
  }

## Error 定義
- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: Step 1 または Step 2
  - Output例: Output パターン2
