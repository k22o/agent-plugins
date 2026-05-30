# claude code 

## コマンド

### 起動

```
claude # 起動
claude -p {指示} # ヘッドレスで (対話的な画面なしで) 処理する
claude --continue # 直前のセッションで開く
claude --resume {sessionID} # 特定のセッションIDで開く
```

### skillなど

```
/model モデルの選択など
/skill skill一覧

```

## claude codeの機能

### context の圧縮

- デフォルトで容量が増えたら圧縮される
- `/clear` コマンドでリセットする

### sub agent

- `.claude/agents/`配下に、agent単位でmdファイルを配置する
- md内にフロントマターを設定することで、メタデータとして管理できる
- mdファイルとyamlファイルでsub agentを使う
- `/agent` コマンドでひな形を作ってから加工するとよい
- sub agentは適宜利用される

### スキル

- `.claude/skills/`配下に、ディレクトリを切って、それぞれ、`SKILL.md`を設定する
- 必要に応じて、その中でさらにファイルを分割できる (`SKILL.md`が親になる)
- `SKILL.md`内にフロントマターを設定することで、メタデータとして管理できる

※ スキルとスラッシュコマンドの違い
- スキルはセッション中すべてに影響を与えうる
- スキルは自動で読みだされる
- スキルは「知識」を保存する

### hook

- ライフライクルの一定のタイミングで必ず実行されるルールを定める

## 参考文献

(参考) https://github.com/GenerativeAgents/claude-code-book


