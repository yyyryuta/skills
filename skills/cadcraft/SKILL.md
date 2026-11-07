---
name: cadcraft
description: Inspect, convert, render, or edit CAD drawings with CADCraft CLI or MCP. Use for DXF/DWG workflows and CADCraft automation; this does not provide automatic architectural component recognition from scanned PDFs.
---

# CADCraftの利用

CAD図形の読込・操作と、柱・梁・壁などの意味の判定を区別する。図形数をそのまま部材数と見なさず、数量・面積には図面の単位、縮尺、ブロック、重複、閉じた境界等の根拠を確認する。

## CLI

`command -v cadcraft-cli`、`cadcraft-cli --version`を確認し、使うサブコマンドの`--help`を読む。サブコマンドによって利用できるオプションが異なる。

```sh
cadcraft-cli info drawing.dxf
cadcraft-cli convert drawing.dxf preview.svg
cadcraft-cli commands
cadcraft-cli mcp
```

- ヘッドレスの調査・変換はCLIを優先し、GUIを開く必要があるかを区別する。
- 読込時の省略・警告、レイヤー・ブロック・図形の情報を調べる。未対応情報が保存時に失われる場合があるため、原本は保持して別ファイルへ出力する。
- 編集は依頼された範囲に限定する。レイヤー名や色だけを建築的な分類の確定根拠にしない。
- プレビューと元のCAD構造の検証を分ける。日本語フォント・寸法・ハッチ・単位・出力範囲も必要に応じて確認する。
- 制御要求がタイムアウトした場合は結果不明と未実行を区別し、状態を調べず編集を再送しない。MCPによるヘッドレス操作とGUIアプリへの接続を混同しない。

## 導入・接続時だけ参照

個人fork：[yyyryuta/cadcraft](https://github.com/yyyryuta/cadcraft)。公式：[README](https://github.com/storytold/cadcraft)、[制御プロトコル](https://github.com/storytold/cadcraft/blob/main/docs/control-protocol.md)。対象OS/CPU用の配布物・チェックサム・ライセンス・フォントの条件を確認する。Mac用バイナリをUbuntuへコピーせず、Linux版を別途導入する。MCP登録・常駐・更新は依頼された場合に行う。
