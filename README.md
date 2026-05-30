# claude-plugins

Claude Code プラグイン集。
その他、claudeの設定に関するバックアップやai-codingの諸々を保存しておく。

- TODO
    - agentの設定
    - ghコマンド連携


## プラグイン一覧

| プラグイン | 説明 |
|-----------|------|
| [code-review](./code-review/) | コードレビュー・セキュリティ・アクセシビリティの統合レビュースキル |


## インストール

### 1. ローカルマーケットプレイスとして登録

このリポジトリを Claude Code のマーケットプレイスとして一度だけ登録する。

```bash
claude plugin marketplace add /mnt/c/Users/kanat/works/tools/claude-plugins
```

### 2. プラグインをインストール

```bash
claude plugin install code-review
```

### アンインストール

```bash
claude plugin uninstall code-review
```
