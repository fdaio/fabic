[简体中文](README.md) · [English](README.en.md) · [日本語](README.ja.md)

> これは中国語の原文 [`README.md`](README.md) の翻訳です。内容が異なる場合は中国語版が優先されます。

# マニュアル索引

このディレクトリは役割マニュアルとレビュー詳細を保存します。入口のルールはルートの [`../AGENTS.ja.md`](../AGENTS.ja.md) にあります。

| ドキュメント | 役割 |
|------|------|
| `../AGENTS.ja.md` | 役割、引き継ぎ、リスク、マージの必須ルール |
| `agent/amy-manual.ja.md` | Amy: 要件の明確化、6 項目のセルフチェック、リスク分類、3 回失敗後の判断 |
| `agent/eva-manual.ja.md` | Eva: 環境受け入れ、チェックコマンド、状態、失敗時の処理 |
| `agent/con-manual.ja.md` | Con: 受け入れテスト、レビュー、PR、CI、マージ、worktree 削除 |
| `agent/review-manual.ja.md` | 階層レビュー、セキュリティレビュー、リスク承認、ロールバック、振り返り |
| `../REVIEW.ja.md` | このハンドブックリポジトリのレビュー重点と高リスク経路 |

## 読む順序

1. 新しいメンバー: `../README.ja.md` -> `../AGENTS.ja.md` -> 役割マニュアル。
2. タスクを引き受ける: `agent/amy-manual.ja.md` -> `../AGENTS.ja.md` の Bob の節 -> `agent/con-manual.ja.md`。
3. ハンドブックを変更する: まず `../REVIEW.ja.md` を読み、`agent/review-manual.ja.md` に従ってセルフチェックする。

## 取り決め

- 例ではプレースホルダーのみを使います: `<project>`、`<repo>`、`<default-branch>`、`REQ-001`、`example.com`。
- ハンドブックは特定のプロジェクト、リポジトリ、組織、ツール、ローカルパスに縛られません。
- ルールが矛盾する場合は `../AGENTS.md` が優先されます。役割マニュアルは操作の詳細を補足するだけです。
- タスクデータ、顧客データ、認証情報、ローカル環境情報をこのリポジトリに書いてはいけません。
