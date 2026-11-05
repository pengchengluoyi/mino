# mino

给 AI agent 用的 Mino 技能（Agent Skills 标准：每个技能一个目录，入口是 `SKILL.md`）。

| 技能 | 一句话 |
|---|---|
| `overview` | Mino 是什么，Scout / Nexus / Workbench 各管什么，Scout 怎么装 |
| `onboarding` | 第一次使用：注册登录、用 OAuth 接入 MCP、装 Scout 节点、准备项目与脚本仓库 |
| `repo-config` | 脚本仓库怎么组织，`config.yaml` 怎么配应用和环境，改了怎么同步给 Nexus |
| `write-test-cases` | 为用例写脚本（Scout 侧）：`app` / `case.scout` / `case.nexus` 怎么用 |
| `check-test-cases` | 写完用 `mino-scout check` 自检，按提示改 |
| `run-test-cases` | 通过 Nexus 执行用例、选设备、看结果、在对话里汇报 |
| `update-skills` | 更新技能；技能和实际对不上时怎么反馈 |

每个技能只做一件事。推荐顺序：`overview` → `onboarding` → `repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。

## 安装给 agent

agent 读取的是 `~/.claude/skills/` 里的技能目录，每个目录里有自己的 `SKILL.md`。

> 请安装 Mino 的技能：
> 1. 把 `https://github.com/pengchengluoyi/mino.git` clone 到 `~/.mino/skills-src`（已存在就 `git pull --ff-only`）；
> 2. 把 `skills/` 下的每个技能目录复制到 `~/.claude/skills/`（已有同名目录时先告诉我，不要覆盖）。同时复制 `VERSION` 和 `CHANGELOG.md` 到 `~/.claude/skills/`；
> 3. 确认 `~/.claude/skills/<技能名>/SKILL.md` 存在，告诉我装到了哪里，并提醒我新开一个会话让技能生效；
> 4. 然后按技能 `onboarding` 带我完成注册、OAuth 接入 MCP 和 Scout 节点。

```bash
git clone https://github.com/pengchengluoyi/mino.git ~/.mino/skills-src
mkdir -p ~/.claude/skills
cp -R ~/.mino/skills-src/skills/. ~/.claude/skills/
cp ~/.mino/skills-src/VERSION ~/.mino/skills-src/CHANGELOG.md ~/.claude/skills/
```

其他 agent 工具把 `~/.claude/skills` 换成它读取技能的目录。

## 更新

```bash
git -C ~/.mino/skills-src pull --ff-only
rm -rf ~/.claude/skills/check-test-cases ~/.claude/skills/onboarding ~/.claude/skills/overview ~/.claude/skills/repo-config ~/.claude/skills/run-test-cases ~/.claude/skills/update-skills ~/.claude/skills/write-test-cases
cp -R ~/.mino/skills-src/skills/. ~/.claude/skills/
cp ~/.mino/skills-src/VERSION ~/.mino/skills-src/CHANGELOG.md ~/.claude/skills/
```

复制完成后新开会话生效。每个版本需要的 Scout / Nexus 版本见 `~/.claude/skills/CHANGELOG.md`。技能有误或缺内容时，见技能 `update-skills`。

## 前提

- 本机已安装并运行 MinoScout（怎么装见 `overview`）；
- agent 已连接 Nexus 的 MCP（`agent_guide` 是总入口）；
- `write-test-cases` 里的方法以本机安装的 `mino_scout/runtime/script_exec.py` 为准。

## 目录规范

- 目录名等于 `SKILL.md` 里的 `name`，小写字母、数字、连字符；
- `SKILL.md` 文件名全大写；
