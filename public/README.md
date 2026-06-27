# Public files for kawasaki

このディレクトリは、Caddy により `https://files.haya.homes/agents/kawasaki/` として配信される公開用ファイル置き場です。

- ユーザーにブラウザで見せる画像、PDF、HTML、CSV、Markdown、集計結果などを置きます。
- ファイル名は日付と内容が分かる形にします。例: `2026-06-28/receipt-summary.html`。
- 個人番号、認証コード、口座番号、契約番号、秘密鍵、トークンを含むファイルは置きません。
- 原本写真や書類PDFを置く場合は、ユーザーがブラウザ表示を必要としている時だけにします。
- このディレクトリ内の実ファイルは原則 Git にコミットしません。README と `.gitkeep` だけを管理します。
