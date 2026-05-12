# aris-for-management

为**创新创业管理**领域(目标 FT50 / UTD24 期刊)定制的 Claude Code 文献综述与论文写作工作流。

基于 [ARIS (Auto-Research-In-Sleep)](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) 的 skill 架构,针对管理学论文(理论贡献 > 实证创新)和中英文双语场景做了适配。

---

## 目录结构

```
aris-for-management/
├── skills/              # 我自己的(可改的)skill 副本
│   ├── research-wiki/      # 跨会话持久记忆
│   ├── research-lit/       # 文献检索(适配 WoS / Scopus / SSRN)
│   ├── paper-plan/         # 论文结构规划
│   └── auto-review-loop/   # 跨模型对抗评审(AMJ 审稿人视角)
├── upstream/            # ARIS 原版(submodule,只读参考)
├── templates/           # 我的模板(论文提取表、理论对比矩阵等)
├── memory/              # 研究 wiki(已读文献、理论笔记、审稿人偏好)
├── docs/                # 我的使用笔记
└── .claude/             # 本地 Claude Code 配置(部分 gitignored)
```

`~/.claude/skills/<name>` → 软链到 `skills/<name>`,所以编辑 repo 里的 SKILL.md,Claude Code 立刻就能用到。

---

## 快速使用

### 装好之后(已完成)

```bash
git clone git@github.com:coujasmine/aris-for-management.git
cd aris-for-management
git submodule update --init upstream
# 软链 ~/.claude/skills/<name> 见 docs/setup.md
```

### 在 Claude Code 里(用斜杠命令调 skill)

```
/research-wiki init                    # 第一次:初始化跨会话记忆
/research-lit "多个大股东 股权操纵"      # 文献检索
/paper-plan                            # 启动论文大纲
/auto-review-loop                      # 让 GPT 模拟 AMJ 审稿人挑你稿子
```

---

## 跨模型对抗评审(Codex MCP)

`.mcp.json` 已配置好 Codex 作为对抗审稿人。你的本机需要:

```bash
brew install codex
codex login   # 用 ChatGPT 账号 或 OpenAI API key
# 然后在项目目录重启 Claude Code,它会自动加载 .mcp.json
```

详见 [docs/setup.md](docs/setup.md)。

---

## 同步 ARIS 上游更新

```bash
git submodule update --remote upstream
git add upstream && git commit -m "update: bump ARIS upstream"
```

我自己的 skill 在 `skills/` 下,不受上游更新影响。需要新功能时手动从 `upstream/skills/<name>` 复制过来再改。

---

## License & 致谢

- 本仓库的定制内容:MIT
- `upstream/`:遵循 ARIS 原项目协议(MIT),感谢 [@wanshuiyin](https://github.com/wanshuiyin)
- `skills/` 下从 ARIS 复制的 skill 保留原作者署名
