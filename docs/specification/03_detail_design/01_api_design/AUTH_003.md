# AUTH_003 セッション確認API
- Method: GET
- Path: /api/auth/me
- 認証: 要（セッション必須）

## Input
- なし（セッションCookieで認証）

## Validation Rule
- なし（入力項目なし）

### チェック順序
- なし

## Process
- Step 1: セッション認証
  - 保護対象パスのためセッション確認を実施
  - 条件: セッションが存在し、SESSION_USER が保存されている
  - NG: E-401-UNAUTHORIZED を返して終了

- Step 2: レスポンス生成
  - セッション内ユーザー情報（id/name/role）を返却

## Output
### パターン1: 成功
- 型: JSON
- HTTP: 200
- 例:
  {
    "id": "5b8e7e4a-4f21-4d0e-b72f-8d5a2d5c9f3b",
    "name": "user001",
    "role": "USER"
  }

### パターン2: 認証エラー（セッションなし/期限切れ）
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-UNAUTHORIZED",
    "message": "セッションユーザーが見つかりません。",
    "details": null,
    "operation": "fetch"
  }

### パターン3: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-UNEXPECTED",
    "message": "予期しないエラーが発生しました。",
    "details": null,
    "operation": "fetch"
  }

## Error 定義
- E-401-UNAUTHORIZED
  - 発生条件: セッションが存在しない、または SESSION_USER が取得できない、または絶対有効期限超過
  - 検出Step: Step 1
  - Output例: Output パターン2

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: Step 1 または Step 2
  - Output例: Output パターン3
