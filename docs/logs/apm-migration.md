# このリポジトリの APM (Agent Package Manager) 対応化手順

対象: Microsoft [`apm`](https://github.com/microsoft/apm)（Agent Package Manager）
参考: [公式ドキュメント](https://microsoft.github.io/apm/)

## 0. APM とは何か（前提知識）

`apm` は AI エージェント向けの設定（skills / prompts / instructions / agents / hooks / plugins / MCP servers）を、
npm や pip のように **宣言的マニフェスト（`apm.yml`）+ ロックファイル（`apm.lock.yaml`）** で管理する OSS ツール。
`apm install` を実行すると、依存関係を解決したうえで GitHub Copilot・Claude Code・Cursor・OpenCode・Codex・Gemini・Windsurf・Kiro など
複数の「harness（実行環境）」向けに設定ファイルを展開してくれる。

このリポジトリ（`agent-plugins`）は Claude Code 専用のプラグイン集（`.claude-plugin/marketplace.json` + `code-review` プラグイン）だが、
APM に対応させることで以下が可能になる。

- `apm install k22o/agent-plugins` のような形で、他ツール（Copilot 等）からもこのリポジトリのスキルを再利用できる
- `apm.lock.yaml` によるバージョン固定・再現性の担保
- 将来 skills/agents/MCP サーバーを増やす際、`.apm/` 配下に集約して一元管理できる

### 用語

| 用語 | 意味 |
|---|---|
| primitive | instructions / skills / prompts / agents / hooks / commands / plugins / MCP servers の総称 |
| harness | 実行環境（Claude Code, Copilot, Cursor 等） |
| `.apm/` | harness 非依存の primitive を置く標準ディレクトリ |
| pack | `apm pack` でプラグイン成果物（zip や `plugin.json` 一式）を生成すること |

## 1. 現状のリポジトリ構成の確認

```
.claude-plugin/marketplace.json   # マーケットプレイス定義（plugins: [code-review]）
claude-buckup/                    # 個人のClaude設定バックアップ（プラグインではない）
code-review/
  .claude-plugin/plugin.json      # プラグイン定義
  skills/
    review/SKILL.md
    review-acessibility/SKILL.md
    review-security/SKILL.md
docs/
```

APM の [Repo shapes](https://microsoft.github.io/apm/producer/repo-shapes) には3パターンある。

1. **Single-plugin** — 1 リポ = 1 プラグイン = 1 マーケットプレイスエントリ
2. **Aggregator** — ローカルにプラグイン実体を持たず、他リポの参照だけを集約するマーケットプレイス専用リポ
3. **Monorepo-hybrid** — 1 リポに複数プラグイン。各プラグインが独立した `apm.yml` / `.apm/` を持つ

README の TODO（agent の設定、gh コマンド連携）から今後プラグインが増える前提のため、**Monorepo-hybrid 形状**を採用する。
現状は `code-review` 1 つだが、将来 `claude-buckup` 相当の設定や新プラグインを追加してもそのまま拡張できる。

## 2. 事前準備

```bash
# APM CLI のインストール（要 Node.js。詳細は公式README参照）
curl -sSL https://aka.ms/apm-unix | sh   # ※パッケージ名は公式リポジトリのインストール手順で必ず確認すること
apm --version
```

> ⚠️ 執筆時点（2026-07）でインストール方法がリポジトリ更新により変わっている可能性がある。
> 実行前に必ず https://github.com/microsoft/apm の README / Releases を確認すること。

## 3. ルートに APM マニフェストを作成

リポジトリ直下（`.claude-plugin/marketplace.json` と同階層）に `apm.yml` を作成し、
マーケットプレイス全体の定義とする。

```bash
apm init -y agent-plugins
```

生成された `apm.yml` を以下のように編集する（既存 `marketplace.json` の内容を移植するイメージ）。

```yaml
name: agent-plugins
version: 1.0.0
description: k22o のプライベート Claude Code プラグイン集
author: k22o
dependencies:
  apm: []
  mcp: []
includes: auto
scripts: {}

# マーケットプレイスとして公開する場合のブロック（apm pack / marketplace generate が参照）
marketplace:
  name: k22o-plugins
  packages:
    - name: code-review
      source: ./code-review
```

> ⚠️ `marketplace:` ブロックの正確なキー名は `apm marketplace generate` の実装に依存するため、
> `apm init` 直後に生成される雛形コメントと [Reference: CLI Commands](https://microsoft.github.io/apm/reference/cli/) を必ず突き合わせて修正すること。

## 4. `code-review` プラグインを APM パッケージ形状に整える

実機検証の結果、`.claude-plugin/plugin.json` は APM の認識に一切不要と判明した（詳細は本文末尾の検証結果を参照）。
`code-review/apm.yml` に `targets: [claude]` と `includes: auto` を持たせるだけで、
`apm install owner/repo/code-review --target claude` が `skills/<name>/SKILL.md` をそのまま検出・展開する。

```
code-review/
├── apm.yml     # targets: [claude] / includes: auto。唯一の手編集対象
└── skills/
    ├── review/SKILL.md
    ├── review-acessibility/SKILL.md
    └── review-security/SKILL.md
```

`.claude-plugin/plugin.json`・`.claude-plugin/marketplace.json`（ルート）はどちらも **Claude Code 純正の
`claude plugin marketplace add` / `claude plugin install` コマンド専用**のファイルであり、
APM 側（`apm install` / `apm pack` の依存解決）からは参照されない。
この構成では APM 導線のみをサポートする方針としたため、両ファイルは削除して構わない（このリポジトリでは削除済み）。
将来 Claude Code 純正コマンドでの配布も必要になった場合は、`apm pack` を実行すれば
`.claude-plugin/plugin.json` / `.claude-plugin/marketplace.json` をいつでも再生成できる。

## 5. コンパイル・検証

`apm compile` / `apm validate` / `apm preview` は **消費側**（`.apm/` ディレクトリを持つプロジェクトや `scripts:` を持つプロジェクト）向けのコマンドで、
今回のようなプラグイン制作側（producer）リポジトリには適用されない（`apm validate` という単独コマンドは存在しない）。
producer 側の妥当性検証には代わりに以下を使う。

```bash
apm marketplace check --offline   # ルート apm.yml の marketplace: ブロックの整合性確認
apm pack --dry-run                # 各パッケージ（code-review/）が正しく pack できるかの確認
```

エラーが出た場合は `apm.yml` の `dependencies` / `includes` / primitive の frontmatter（name, description）を見直す。

## 6. Claude Code 向けにローカルインストールして動作確認

```bash
apm install ./code-review --target claude
```

- `.claude/skills/` 配下にファイルが生成されることを確認する
- 生成物と、既存の `code-review/skills/*/SKILL.md` の内容が一致する／劣化していないことを目視確認する（`diff -r` 推奨）
- `apm_modules/` は自動で `.gitignore` に追加されるので、コミット対象に含めない
- 検証後、テスト用に生成された `apm_modules/`・`.claude/skills/`・ルート `apm.yml` に追記された `dependencies:` ブロック・`apm.lock.yaml` は削除して元に戻す（このリポジトリ自身が `code-review` に依存する必要はないため）

## 7.（任意）プラグイン成果物のパック（配布用）

本リポジトリは APM 導線に一本化し `.claude-plugin/` を持たない方針としたため、通常はこの手順は不要。
Claude Code 純正コマンド（`claude plugin install` 等）向けに配布物が必要になった場合のみ実施する。

```bash
cd code-review
apm pack
```

`build/code-review-<version>/` 配下に `plugin.json`（と、依存があれば `apm.lock.yaml`）が生成される。
`build/` は `.gitignore` 対象なのでコミットしない。

複数プラグインをまとめたマーケットプレイス索引が必要になった場合（プラグインが増えたら）:

```bash
apm pack   # ルート apm.yml。marketplace: ブロックのみの場合、.claude-plugin/marketplace.json を生成する
```

> ⚠️ `apm marketplace generate` というコマンドは apm CLI v0.25.0 には存在しない。上記の通り `apm pack` が実体。

## 8. コミット対象の整理

| パス | コミット | 備考 |
|---|---|---|
| `apm.yml`（ルート・各プラグイン） | ✅ | マニフェスト（唯一の手編集対象） |
| `apm.lock.yaml` | ✅ | ロックファイル |
| `.apm/` | ✅ | harness 非依存 primitive |
| `.claude-plugin/`（ルート・各プラグイン） | ❌（削除済み） | APM の依存解決・pack には一切不要と実機確認済み。Claude Code 純正の `claude plugin marketplace add` / `claude plugin install` 専用ファイルのため、その導線を案内しない本リポジトリでは削除した |
| `apm_modules/` | ❌ | `.gitignore` に自動追加されるキャッシュ |
| `build/`（pack 成果物） | ❌ | `.gitignore` 対象。`apm pack` を実行すればいつでも `.claude-plugin/plugin.json` 込みで再生成できる |

> 方針: **APM 導線に一本化**する。README や利用者向け案内では `claude plugin marketplace add` / `claude plugin install` は案内せず、
> `apm install` のみを唯一のインストール手段とする。そのため `.claude-plugin/*.json` はリポジトリに置かない。
> 将来 Claude Code 純正コマンドでの配布も必要になった場合のみ、`apm pack` で都度生成すればよい（`apm.yml` があれば再生成可能）。

## 9. README / ドキュメント更新

`README.md` のインストール手順を APM 経由のみに一本化する（`claude plugin marketplace add` / `claude plugin install` の手順は案内しない）。

```bash
# 1. apm CLI をインストール
curl -sSL https://aka.ms/apm-unix | sh

# 2. code-review パッケージを直接インストール（owner/repo/subdir 形式）
apm install k22o/claude-plugins/code-review --target claude

# アンインストール
apm uninstall code-review
```

> 実際の GitHub リモートは `k22o/claude-plugins`（`agent-plugins` ではない。`git remote -v` で要確認）。
> また `code-review` はルート直下ではなくサブディレクトリのパッケージなので、
> `owner/repo` だけでなく `owner/repo/code-review` まで指定する必要がある
> （`owner/repo` だけだとルートの `marketplace:` ブロックしか解決されず、スキル自体は展開されない）。

## 10. （任意）CI での自動 pack / 検証

[`microsoft/apm-action`](https://github.com/microsoft/apm-action)（GitHub Action）を使うと、
push / PR 時に `apm compile` → `apm pack` を自動実行し、壊れた primitive を検知できる。
`.github/workflows/apm.yml` を新規作成して組み込む（既存 CI が無ければ新設）。

## 未確定・要検証事項まとめ（検証結果: 2026-07-18, apm CLI v0.25.0）

1. `apm` CLI の正確なインストールコマンド（npm パッケージ名・配布形態）
   → ✅ 検証環境には既に `/usr/local/bin/apm` (v0.25.0) が導入済みだった。インストール手順自体は未検証のまま。
2. `apm.yml` の `marketplace:` ブロックの正式なキー構造
   → ✅ 確認済み。`apm marketplace init` / `apm marketplace package add` で生成される実際の構造は以下の通り（本文の例とは異なる）。
      ```yaml
      marketplace:
        owner:
          name: k22o
          url: https://github.com/k22o
        build:
          tagPattern: "v{version}"
        outputs:
          claude: {}
        packages:
          - name: code-review
            description: ...
            source: ./code-review
            ref: <commit-sha>   # ローカルパスの場合、ネットワーク接続なしでは HEAD を自動解決できないため明示 SHA が必須
      ```
      `marketplace.name` のようなトップレベルキーは存在せず、パッケージ名は `apm.yml` 直下の `name:`（リポジトリ全体の名前）を使う。
3. `plugin.json` を `.claude-plugin/` 配下に置いたままで APM の Plugin Collection として認識されるか
   → ✅ 認識される。`apm pack`（`code-review/apm.yml` に `targets: [claude]` を設定した状態）は
      `.claude-plugin/plugin.json` を正しく読み書き対象として認識した。`.apm/` への移行は不要だった。
4. `apm marketplace generate` の出力パスが既存の `.claude-plugin/marketplace.json` とそのまま置換可能か
   → ⚠️ `apm marketplace generate` というコマンドは v0.25.0 に存在しない（`Error: No such command 'generate'`）。
      代わりに、ルート `apm.yml` に `marketplace:` ブロックのみを置いた状態で `apm pack` を実行すると、
      `.claude-plugin/marketplace.json` へ書き込まれることを `--dry-run` で確認済み（パスは既存ファイルと完全一致）。
      本文中の「7. プラグイン成果物のパック」「`apm marketplace generate`」の記述は `apm pack` に読み替えること。
5. `apm publish` / `apm marketplace publish` は非推奨化が進行中（`apm pack` + `apm marketplace generate` が現行の推奨経路）
   → 未検証（今回は実行していない）。`apm pack --help` の文面上は `apm pack` が現行の推奨経路である点のみ確認。
6. （新規判明）本文中のインストール例 `apm install k22o/agent-plugins --target claude` はリポジトリ名が誤り。
   → 実際の GitHub リモートは `k22o/claude-plugins`（`git remote -v` で確認）。また `code-review` はサブディレクトリパッケージのため、
      `apm install k22o/claude-plugins/code-review --target claude` のように owner/repo/subdir 形式で指定する必要がある
      （`k22o/claude-plugins` だけではリポジトリ直下の `marketplace:` ブロックしか解決されず、スキル自体は展開されない）。
      これは実際に `--dry-run` で GitHub 上の `code-review/.claude-plugin/plugin.json@main` を解決できることまで確認済み。
7. （新規判明）`apm compile` / `apm validate` / `apm preview`（本文 5 節）は消費側（`.apm/` を持つプロジェクト）向けのコマンドで、
   今回のようなプラグイン制作側（`marketplace:` ブロックのみを持つ producer リポジトリ）には適用されない。
   → `apm validate` という単独コマンドは存在しない。妥当性検証には代わりに `apm marketplace check --offline` と
      `apm pack --dry-run` を使うこと。

## 参考リンク

- https://github.com/microsoft/apm
- https://microsoft.github.io/apm/
- https://microsoft.github.io/apm/quickstart/
- https://microsoft.github.io/apm/concepts/what-is-apm/
- https://microsoft.github.io/apm/reference/package-types/
- https://microsoft.github.io/apm/producer/repo-shapes
- https://microsoft.github.io/apm/getting-started/first-package/
- https://github.com/microsoft/apm-action
