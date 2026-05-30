# code-review

コードレビュー・セキュリティレビュー・アクセシビリティレビューを行う Claude Code プラグイン。

## スキル一覧

| スキル | 説明 |
|--------|------|
| `review` | PR をシニアエンジニアレベルで厳密にコードレビューし、リリース可否を判断する |
| `review-security` | OWASP Top 10 準拠のセキュリティ観点でレビューし、問題を指摘・修正する |
| `review-acessibility` | WCAG 2.1 準拠のアクセシビリティ観点でレビューし、問題を指摘・修正する |

## 使い方

### review（統合コードレビュー）

PR番号を指定して実行する。`review-security` と `review-acessibility` を自動的に呼び出してサブエージェントで並列実行する。

```
/review <PR番号> [仕様書URL] [FigmaURL]
```

### review-security（セキュリティレビュー単体）

```
/review-security [ファイルパスまたはディレクトリ]
```

### review-acessibility（アクセシビリティレビュー単体）

```
/review-acessibility [ファイルパスまたはディレクトリ]
```

## インストール

親ディレクトリ（`claude-plugins/`）をマーケットプレイスとして登録してからインストールする。

```bash
# マーケットプレイス登録（初回のみ）
claude plugin marketplace add /mnt/c/Users/kanat/works/tools/claude-plugins

# プラグインをインストール
claude plugin install code-review
```

### アンインストール

```bash
claude plugin uninstall code-review
```
