# Macの導入記録とUbuntuへの再現手順

確認日：2026-11-07。対象は利用者のMac（Apple Silicon / arm64）。Ubuntuへは今回接続・導入していない。同じGitHub・Codexアカウントでも、実行ファイル・Skill・依存パッケージ・MCP設定は自動同期されない。

今回の導入は開発・検証用ツールの準備。RustXNGやCADCraftをViewer本番の必須依存に変更したものではなく、図面解析の認識精度も未検証。

## 1. リポジトリ

| 対象 | 個人fork | 元リポジトリ |
|---|---|---|
| Skills | https://github.com/yyyryuta/skills | anthropics/skills |
| RustXNG | https://github.com/yyyryuta/RustXNG | sahenjp/RustXNG |
| CADCraft | https://github.com/yyyryuta/cadcraft | storytold/cadcraft |
| OfficeCLI | https://github.com/yyyryuta/OfficeCLI | iOfficeAI/OfficeCLI |

Skillsの既存forkを利用し、残る3件を新規forkした。実行ファイルは個人forkから独自ビルドせず、元リポジトリの公式リリースを取得した。

## 2. Skills

配置先は`~/.codex/skills/`。

| Skill | 出所・版 | 結果 |
|---|---|---|
| `webapp-testing` | 個人Skills forkの`ca45fc496a3436251f191354091d557da31c15b5` | 新規導入 |
| `frontend-design` | 同上 | 新規導入 |
| `mcp-builder` | 同上 | 新規導入 |
| `rustxng` | 個人Skills forkへ独自手順を追加 | 新規作成・導入 |
| `cadcraft` | 同上 | 新規作成・導入 |
| `skill-creator` | Codex組み込み | 既存を利用。重複導入なし |
| `officecli-docx` | `~/.agents/skills/officecli-docx/` | 既存を利用 |

独自の2つも含めた固定コミットは`c2ca7e821e121dadc3763526f2dca8885e51ca16`。同コミットで上記5ディレクトリを取得できる。

- [RustXNG Skill](https://github.com/yyyryuta/skills/blob/c2ca7e821e121dadc3763526f2dca8885e51ca16/skills/rustxng/SKILL.md)
- [CADCraft Skill](https://github.com/yyyryuta/skills/blob/c2ca7e821e121dadc3763526f2dca8885e51ca16/skills/cadcraft/SKILL.md)

3つのAnthropic製Skillの個別LICENSEはApache-2.0を確認し、付属ファイルごと導入した。Anthropicの文書系Skillは異なる利用条件のため移植せず、OfficeCLI等の既存経路を利用する。

5つのSkillはCodexの`quick_validate.py`で構造検査に合格。判断品質や全ユースケースの検証を意味しない。導入したSkillは次のターンから利用可能。

## 3. 実行ファイル

| ツール | 版 | Macの配置先 | 確認結果 |
|---|---|---|---|
| RustXNG | 0.30.0 | `~/.local/bin/rustxng` | バージョン、起動、製品識別、公開語句の検索を確認 |
| CADCraft CLI | 0.18.0 / `32238e83c2c8bf643994132883e94bc96961f649` | `~/.local/bin/cadcraft-cli` → `~/.local/share/cadcraft/v0.18.0/cli/cadcraft-cli` | サンプル生成、DXF読込、SVG出力を確認 |
| CADCraftアプリ | 0.18.0 | `~/Applications/CADCraft.app` | 配置・Info.plistの版を確認。GUI操作は未確認 |
| OfficeCLI | 1.0.157 | `~/.local/bin/officecli` | 既存。自動更新を抑止して版のみ確認。再インストールなし |

RustXNGのLICENSE・NOTICEは`~/.local/share/rustxng/v0.30.0/`、CADCraft CLIのライセンス・NOTICE・同梱フォント等の表記は`~/.local/share/cadcraft/v0.18.0/cli/`に保持した。取得物のチェックサム一致は配布元の内容との一致であり、別主体による安全性の証明ではない。

| 公式取得物 | SHA-256（一致確認済み） |
|---|---|
| `rustxng-v0.30.0-aarch64-apple-darwin` | `53ad48ba3c98fad64b57f9fcc66b984ea8cb5ff7363991b98dbcc7d86024aece` |
| `cadcraft-cli-0.18.0-macos-universal.zip` | `323d276a2693c57ef6d15719aedf64903391d2a309f7cd1332434fa1b9a03e3e` |
| `cadcraft-0.18.0-macos-universal.dmg` | `e59d98615fcecc9d5ef0650c63486229d7d2aa22914fb8d1050a5d071329f9c9` |

## 4. Python・ブラウザ依存

既存Pythonへ直接パッケージを追加せず、`~/.local/share/agent-tools/venv/`に専用環境を作成した。

```text
PyYAML==6.0.3
playwright==1.64.1
greenlet==3.5.6
pyee==14.0.0
typing_extensions==4.16.0
```

PlaywrightのChromium・Headless Shell（revision 1248）・FFmpeg（1013）を標準キャッシュへ導入した。アプリの画面テストは実施していない。`webapp-testing/scripts/with_server.py --help`の実行は確認済み。実際のWeb検証では使用環境のブラウザ用ルールにも従う。

MCPサーバを新規実装する各プロジェクトのSDKや、`mcp-builder`の任意の評価処理で使うAnthropic/MCP Python SDKは、必要になった作業で別途導入する。今回APIキーや課金を伴う評価は設定・実行していない。

## 5. 動作確認の範囲

- RustXNG：空きポートへループバック限定で一時起動。`/healthz`・`/config`を確認し、Wikipediaエンジンで`PostgreSQL`を検索して3件を取得。確認後に自分で起動したプロセスを終了した。検索品質の比較評価ではない。
- CADCraft：CLIの`sample floorplan`でDXFを作成。`info`で図形16件と読込時省略0件を確認。SVG 21,219 bytesを出力しXMLとして解析できた。画像の目視・実務図面・部材分類・数量精度は未検証。
- CADCraftのDMGは配置後に取り外した。GUI起動、Gatekeeper設定変更、署名の上書き、常駐化は行っていない。
- RustXNG・CADCraftの自動起動、CodexのMCP接続設定は未追加。CLIとSkillを利用可能にした段階。

## 6. Ubuntu側で再現するとき

1. OS・CPU、既存の実行ファイル・サービス・Skillsを確認する。Mac用の実行ファイルやPython仮想環境をコピーしない。
2. 個人Skills forkの上記固定コミットから`skills/webapp-testing`・`skills/frontend-design`・`skills/mcp-builder`・`skills/rustxng`・`skills/cadcraft`をCodexのskill-installerで導入する。既存ディレクトリがある場合は上書きせず差分を確認する。
3. RustXNGは公式の対象リリースのLinux x86_64用配布物を選び、SHA-256・LICENSE・NOTICEを確認して配置する。必要なLinux互換性は公式資料と実機で確認する。
4. CADCraftもLinux用配布物を別途選ぶ。GUIのない環境では、必要なCLIがその配布物に含まれるかと、ヘッドレスで動くかを確認する。Macアプリが動いたことをLinuxの動作根拠にしない。
5. OfficeCLIが必要なら既存版を確認し、未導入の場合だけLinux版を導入する。文書生成はViewer側のテンプレート運用手順に従う。
6. 画面テストが必要になったらLinux上でPython環境・Playwright・ブラウザとOS依存を用意する。導入と、実際の画面テスト合格を区別する。
7. 起動・製品識別・実検索、原本のコピーによるCAD読込・変換を確認して記録する。常駐化・MCP設定・Viewerへの統合は、その範囲を決めてから行う。

自動修正・自己改善ループ、T3 Code・Herder・Orca・OpenShell・NemoClaw・GraphRAG・Dreamingの導入は今回の対象外。研究論文は設計知見として保存し、実行ソフトウェアとして導入しない。
