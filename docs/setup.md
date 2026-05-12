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

## 跨模型评审(待配置)

ARIS 的 `auto-review-loop` 默认调用 Claude 自审 → 容易陷入"自圆其说"。
真正有价值的设置是:Claude 写,**另一家模型**(Codex/GPT 系、Gemini、Kimi)读 + 挑刺。

两种接法,见 `docs/mcp-codex.md`(待补):

1. **Codex CLI + MCP bridge**(推荐英文 / FT50 审稿模拟)
2. **Kimi / GLM API**(推荐中文文献处理,API 便宜)

## 同步另一台机器

```bash
git clone git@github.com:coujasmine/aris-for-management.git
cd aris-for-management
git submodule update --init upstream
# 然后跑上面的软链脚本
```
