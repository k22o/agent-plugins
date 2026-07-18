# agent-plugins

agent用のplugin集。

その他、バックアップやai-codingの諸々を保存しておく。

- TODO
    - agentの設定
    - ghコマンド連携


## プラグイン一覧

| プラグイン | 説明 |
|-----------|------|
| [code-review](./code-review/) | コードレビュー・セキュリティ・アクセシビリティの統合レビュースキル |


## MCPサーバー一覧

| MCPサーバー | transport | 用途 |
|-----------|-----------|------|
| context7 | http | ライブラリドキュメント参照 |
| chrome-devtools | stdio (`npx chrome-devtools-mcp`) | Chrome DevTools操作 |


## インストール

[APM (Agent Package Manager)](https://microsoft.github.io/apm/) 経由でインストールする。
プラグイン・MCPサーバーの依存関係は [apm.yml](./apm.yml) に定義済み。

### 1. apm CLI をインストールする

```bash
curl -sSL https://aka.ms/apm-unix | sh
apm --version
```

### 2. 依存関係一式（プラグイン + MCPサーバー）をインストール

```bash
apm install k22o/claude-plugins --target claude
```

このリポジトリを既に clone 済み（作業中のこのディレクトリ含む）であれば、
リポジトリ直下で引数なしで実行してもよい。

```bash
apm install --target claude
```

プラグインだけを単体でインストールしたい場合:

```bash
apm install k22o/claude-plugins/code-review --target claude
```
