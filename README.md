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


## インストール

[APM (Agent Package Manager)](https://microsoft.github.io/apm/) 経由でインストールする。

### 1. apm CLI をインストールする

```bash
curl -sSL https://aka.ms/apm-unix | sh
apm --version
```

### 2. プラグインをインストール

```bash
apm install k22o/agent-plugins/code-review --target claude
```

### アンインストール

```bash
apm uninstall code-review
```
