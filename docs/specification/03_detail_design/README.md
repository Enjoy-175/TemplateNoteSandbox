# 詳細設計

API・処理・DBの具体設計をまとめるディレクトリです。

- 01_api_design/: API仕様。
  - 01_api_list.md: API一覧。
  - AUTH_*.md / USR_*.md / CAT_*.md / TAG_*.md / THEME_*.md / NOTE_*.md: 機能別のAPI詳細。
  - images/: API関連の画像。
- 02_process_design/: 処理設計。
  - 01_sequence_diagram/: 代表機能のシーケンス整理と関連画像。
  - 02_process_flow/: 機能別の処理フローと関連画像。
- 03_db_design/: DB設計。
  - 01_er_diagram.md: ER全体の整理。
  - 02_users.md 〜 09_note_tags.md: テーブル定義。
  - images/: DB関連の画像。

```
docs/specification/03_detail_design
├── 01_api_design/
│   ├── 01_api_list.md
│   ├── AUTH_001.md
│   ├── AUTH_002.md
│   ├── AUTH_003.md
│   ├── CAT_001.md
│   ├── CAT_002.md
│   ├── CAT_003.md
│   ├── CAT_004.md
│   ├── CAT_005.md
│   ├── CAT_006.md
│   ├── NOTE_001.md
│   ├── NOTE_002.md
│   ├── NOTE_003.md
│   ├── NOTE_004.md
│   ├── NOTE_005.md
│   ├── NOTE_006.md
│   ├── NOTE_007.md
│   ├── TAG_001.md
│   ├── TAG_002.md
│   ├── TAG_003.md
│   ├── TAG_004.md
│   ├── TAG_005.md
│   ├── TAG_006.md
│   ├── THEME_001.md
│   ├── THEME_002.md
│   ├── THEME_003.md
│   ├── THEME_004.md
│   ├── THEME_005.md
│   ├── USR_001.md
│   ├── USR_002.md
│   ├── USR_003.md
│   └── images/
├── 02_process_design/
│   ├── 01_sequence_diagram/
│   │   ├── main_screen_function.md
│   │   └── images/
│   └── 02_process_flow/
│       ├── 01_process_flow_list..md
│       ├── AUTH_001.md
│       ├── AUTH_002.md
│       ├── AUTH_003.md
│       ├── CAT_001.md
│       ├── CAT_002.md
│       ├── CAT_003.md
│       ├── CAT_004.md
│       ├── CAT_005.md
│       ├── NOTE_001.md
│       ├── NOTE_002.md
│       ├── NOTE_003.md
│       ├── NOTE_004.md
│       ├── NOTE_005.md
│       ├── NOTE_006.md
│       ├── NOTE_007.md
│       ├── TAG_001.md
│       ├── TAG_002.md
│       ├── TAG_003.md
│       ├── TAG_004.md
│       ├── TAG_005.md
│       ├── THM_001.md
│       ├── THM_002.md
│       ├── THM_003.md
│       ├── THM_004.md
│       ├── THM_005.md
│       ├── USR_001.md
│       ├── USR_002.md
│       ├── USR_003.md
│       └── images/
├── 03_db_design/
│   ├── 01_er_diagram.md
│   ├── 02_users.md
│   ├── 03_categoriese.md
│   ├── 04_tags.md
│   ├── 05_template_themes.md
│   ├── 06_template_questions.md
│   ├── 07_notes.md
│   ├── 08_note_answers.md
│   ├── 09_note_tags.md
│   └── images/
└── README.md
```
