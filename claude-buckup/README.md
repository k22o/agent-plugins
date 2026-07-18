# backup

PC上のclaudeの設定をバックアップする(backup deirectory)。
再利用する際には、以下のコマンドで `~/.claude/` にコピーする。

```bash
cp settings.json ~/.claude/settings.json && \
cp CLAUDE.md ~/.claude/CLAUDE.md && \
cp statusline.sh ~/.claude/statusline.sh && chmod +x ~/.claude/statusline.sh && \
mkdir -p ~/.claude/hooks && \
cp hooks/protect-files.sh ~/.claude/hooks/protect-files.sh && chmod +x ~/.claude/hooks/protect-files.sh
```

## 環境構築

### コマンドの設定

```
$ sudo apt install jq
```

### MCPの設定

```
$ claude mcp add --transport http context7 https://mcp.context7.com/mcp
$ claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```
