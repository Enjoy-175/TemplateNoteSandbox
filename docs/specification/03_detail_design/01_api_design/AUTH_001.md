# AUTH_001 ログインAPI
- Method: POST
- Path: /api/auth/login
- 認証: 不要

## Input
- name（文字列, 必須, 1〜16文字, 空白のみ不可, 例: "user001"）
- password（文字列, 必須, 8〜16文字, 英字・数字・記号を各1文字以上含む, 例: "Passw0rd!"）

## Validation Rule
- Rule V001: ユーザー名の必須チェック
  - 対象: name
  - 条件: null または 空文字 または 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "ユーザー名を入力してください。"

- Rule V002: ユーザー名の文字数チェック
  - 対象: name
  - 条件: 長さ < 1 または 長さ > 16
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "ユーザー名は1〜16文字で入力してください。"

- Rule V003: パスワードの必須チェック
  - 対象: password
  - 条件: null または 空文字 または 空白のみ
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "パスワードを入力してください。"

- Rule V004: パスワードの文字数チェック
  - 対象: password
  - 条件: 長さ < 8 または 長さ > 16
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "パスワードは8〜16文字で入力してください。"

- Rule V005: パスワードの形式チェック
  - 対象: password
  - 条件: 正規表現 ^(?=.*[A-Za-z])(?=.*\d)(?=.*[^A-Za-z0-9]).{8,16}$ に一致しない
  - 判定: NG
  - エラーコード: E-400-VALIDATION
  - メッセージ: "パスワードは英字（a〜z/A〜Z）・数字（0〜9）・記号（!@#$%^&*など）を各1文字以上含む8〜16文字で入力してください。"

### チェック順序
1. V001 → NGなら終了
2. V002 → NGなら終了
3. V003 → NGなら終了
4. V004 → NGなら終了
5. V005 → NGなら終了
6. 全てOKなら次の処理へ

## Process
- Step 1: 入力バリデーション
  - Validation Rule を順に適用
  - NG: E-400-VALIDATION を返して終了

- Step 2: 資格情報の取得
  - ユーザー名で認証情報を検索
  - 条件: 該当ユーザーが存在する
  - NG: E-404-USER-NOT-FOUND を返して終了

- Step 3: パスワード照合
  - 受信パスワードと保存済みハッシュを照合
  - 条件: passwordEncoder.matches(...) が true
  - NG: E-401-PASSWORD-MISMATCH を返して終了

- Step 4: ユーザー取得
  - 認証情報に紐づくユーザーIDでユーザー取得
  - 条件: ユーザーが存在する
  - NG: E-404-USER-NOT-FOUND を返して終了

- Step 5: セッション確立
  - セッションIDを再発行し、ユーザー情報をセッションに保存
  - 成功: Step 6 へ

- Step 6: レスポンス生成
  - ログインユーザー情報（id/name/role）を返却

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

### パターン2: Validation Error（例: V001）
- 型: JSON
- HTTP: 400
- 例:
  {
    "code": "E-400-VALIDATION",
    "message": "ユーザー名を入力してください。",
    "details": [
      { "field": "name", "message": "ユーザー名を入力してください。" }
    ],
    "operation": "create"
  }

### パターン3: 認証エラー（パスワード不一致）
- 型: JSON
- HTTP: 401
- 例:
  {
    "code": "E-401-PASSWORD-MISMATCH",
    "message": "パスワードが間違っています。",
    "details": null,
    "operation": "create"
  }

### パターン4: 業務エラー（ユーザー未存在）
- 型: JSON
- HTTP: 404
- 例:
  {
    "code": "E-404-USER-NOT-FOUND",
    "message": "ユーザーが存在しません。",
    "details": null,
    "operation": "create"
  }

### パターン5: システムエラー
- 型: JSON
- HTTP: 500
- 例:
  {
    "code": "E-500-DB",
    "message": "システムエラーが発生しました。",
    "details": null,
    "operation": "create"
  }

## Error 定義
- E-400-VALIDATION
  - 発生条件: 入力バリデーションNG
  - Input例: name = ""
  - 検出Step: Step 1
  - Output例: Output パターン2

- E-401-PASSWORD-MISMATCH
  - 発生条件: 入力パスワードが保存済みハッシュと一致しない
  - Input例: name = "user001", password = "WrongPass1!"
  - 検出Step: Step 3
  - Output例: Output パターン3

- E-404-USER-NOT-FOUND
  - 発生条件: ユーザー名に該当する認証情報またはユーザーが存在しない
  - Input例: name = "no_user", password = "Passw0rd!"
  - 検出Step: Step 2 または Step 4
  - Output例: Output パターン4

- E-500-DB
  - 発生条件: DBアクセス例外
  - 検出Step: Step 2/Step 4
  - Output例: Output パターン5

- E-500-UNEXPECTED
  - 発生条件: 予期しない例外
  - 検出Step: 全Step
  - Output例:
    {
      "code": "E-500-UNEXPECTED",
      "message": "予期しないエラーが発生しました。",
      "details": null,
      "operation": "create"
    }
