# Setup Notes

## 软链 ~/.claude/skills/

让 Claude Code 能调用 repo 里的 skill:

```bash
REPO="$HOME/Desktop/yiyuan mai/研究项目/AI-R-I-S"
mkdir -p ~/.claude/skills
for s in research-wiki research-lit paper-plan auto-review-loop; do
  ln -s "$REPO/skills/$s" ~/.claude/skills/$s
done
```

之后新增 skill 时,记得补一条同样的软链。

## 跨模型评审(Codex MCP bridge,已配置)

`auto-review-loop` 默认调用 Claude 自审 → 容易"自圆其说"。
正确做法:Claude 写,**另一家模型**(GPT 系)读 + 挑刺。

### 当前配置

repo 根目录的 `.mcp.json` 已经定义了 Codex MCP server:

```json
{
  "mcpServers": {
    "codex": { "command": "codex", "args": ["mcp-server"] }
  }
}
```

### 启用步骤(每台新机器一次)

1. 装 Codex CLI:`brew install codex`(或参考 OpenAI 官方文档)
2. 登录:`codex login` — 用 ChatGPT 账号(Plus/Pro)或 OpenAI API Key
3. 验证:`codex login status` 应该返回 `Logged in using ChatGPT` 或 `Logged in with API key`
4. **重启当前 Claude Code 会话**(`Ctrl+C` 然后重开),让它检测到 `.mcp.json`
5. 第一次启动时,Claude Code 会提示 "Detected MCP servers in project — allow?",选 **yes**
6. 验证:Claude Code 里输入 `/mcp` 查看连接状态,应该看到 `codex` ✓

### 验证调用成功

启用后,在 Claude Code 里跑一个简单测试:

```
请用 mcp__codex__codex 工具问 Codex:"你现在用的什么模型?reasoning effort 多少?"
```

如果返回 Codex 的回答 + 模型版本(应该是 gpt-5.5+),桥就通了。

### 用 Codex 跑跨模型评审

之后想让 Codex 当对抗审稿人,直接在 Claude Code 里:

```
/auto-review-loop "我的稿件路径"
```

skill 会自动调 `mcp__codex__codex`,把 AMJ 审稿视角的 prompt 发给 Codex。

### 备选:Kimi / GLM(中文文献,API 便宜)

如果以后想加第二个对抗审稿人(比如让 Kimi 读中文文献摘要),给 `.mcp.json` 加一个 entry:

```json
"kimi": {
  "command": "npx",
  "args": ["-y", "@moonshot/mcp-kimi"],
  "env": { "MOONSHOT_API_KEY": "${MOONSHOT_API_KEY}" }
}
```

(具体包名以 Moonshot 官方为准,API Key 别提交到 repo)

## 同步另一台机器

```bash
git clone git@github.com:coujasmine/aris-for-management.git
cd aris-for-management
git submodule update --init upstream
# 然后跑上面的软链脚本
```
