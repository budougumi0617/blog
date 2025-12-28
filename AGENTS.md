# Repository Guidelines

## プロジェクト構成とモジュール配置
- `content/post/YYYY/MM/DD/slug.md` に Markdown 記事を配置。slug は lower_snake_case。front matter は `title`, `date`, `draft`, `tags`, `categories`, `keywords`, `author`, `toc`, `twitterImage`, `slug` を整える。
- `static/` は画像や `logos/` などのアセット置き場。公開パスは `/` 直下に展開されるので衝突に注意。
- `layouts/` はテーマ上書き用の Hugo テンプレート。テーマは `themes/hugo-octopress` をサブモジュールで管理。ビルド成果物は `public/` (サブモジュール) に出力され、`deploy.sh` で更新。
- `archetypes/` で記事雛形を管理。`data/` は構造化データ用。`config.toml` がサイト設定の唯一のソース。`aqua.yaml` で `hugo` バージョン固定。

## セットアップ・ビルド・開発コマンド
- `aqua install` または `aqua i` で `hugo` v0.145.0 を取得。`aquaproj` を事前にセットアップ。
- `hugo server -D --disableFastRender` でドラフト含むプレビュー。`http://localhost:1313` を確認し Broken Link などを目視チェック。
- `hugo --gc --minify` で本番相当ビルド。`./deploy.sh "msg"` は `public` サブモジュールを更新し master へ push するデプロイ手順。

## コーディングスタイルと命名規則
- Markdown は 1 行 80-100 文字目安。段落内改行は 2 スペースで意図的に改行。
- 見出しは `#` 階層を崩さず、冒頭に `# タイトル`、本文中の `##` 以降は時間順やトピック順に配置。
- コードブロックは適切な言語ラベルを付与。引用・図版は `static/` のパスを使いローカル参照。
- ファイル名・ディレクトリ名は西暦/ゼロパディング済みの日付を含める。画像は `static/YYYY/MM/DD/` など記事日付配下に置く。

## テストと検証
- 公開前に `hugo server -D` で目視チェック、`draft = true` を `false` に変更し `<!--more-->` を配置して概要を明示。
- ビルド時に警告が出たテンプレートやリソースは修正。`public/` をコミットする前に差分が最小化されているか確認。
- CI は無いのでローカルでビルド成功を確認し、必要なら `hugo --printUnusedTemplates --printI18nWarnings` で未使用テンプレートを検証。

## コミットとプルリクエスト
- コミットメッセージは英語の命令形を推奨例: `Add post about NRQL facets`。記事追加は日付とテーマを含め簡潔に。
- PR は概要、変更範囲 (記事・テンプレート・設定)、ローカルビルド結果を記載。画像差分や `public/` 更新がある場合はスクリーンショットを添付。
- `public/` はサブモジュールなので、ソース変更の PR と公開用コミットを分けるか、`deploy.sh` 実行後のサブモジュール更新を同一 PR に含める。

## セキュリティと設定の注意
- Google Analytics や Disqus の ID は `config.toml` で管理。秘密情報や鍵ファイルをコミットしない。
- `public/` は GitHub Pages 用ブランチを想定。不要なファイルを置かないようにし、`.gitmodules` の変更が必要なときはメンテナと相談。
