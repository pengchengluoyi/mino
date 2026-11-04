# mino

给 AI agent 用的 Mino 技能（Agent Skills 标准：每个技能一个目录，入口是 `SKILL.md`）。

| 技能 | 一句话 |
|---|---|
| `overview` | Mino 是什么，Scout / Nexus / Workbench 各管什么，Scout 怎么装 |
| `onboarding` | 第一次使用：注册登录、生成 MCP token、装 Scout 节点、准备项目与脚本仓库（带 agent 操作 Workbench） |
| `repo-config` | 脚本仓库怎么组织，`config.yaml` 怎么配应用和环境，改了怎么同步给 Nexus |
| `write-test-cases` | 为用例写脚本（Scout 侧）：`app` / `case.scout` / `case.nexus` 怎么用 |
| `check-test-cases` | 写完用 `mino-scout check` 自检，按提示改 |
| `run-test-cases` | 通过 Nexus 执行用例、选设备、看结果、在对话里汇报 |
| `update-skills` | 更新技能；技能和实际对不上时怎么反馈 |

每个技能只做一件事。推荐顺序：`overview` → `onboarding` → `repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。

## 安装给 agent

### 方式一：把下面这段话发给 agent（推荐）

> 请安装 Mino 的技能：
> 1. 把 `https://github.com/pengchengluoyi/mino.git` clone 到 `~/.mino/skills-src`（已存在就 `git pull --ff-only`）；
> 2. 对 `~/.mino/skills-src/skills/` 下的每个技能目录，在你读取技能的目录里建一个**软链接**指向它（Claude Code 是 `~/.claude/skills/`，只想给当前项目用就放项目里的 `.claude/skills/`）；目标位置已有同名目录时先告诉我，不要覆盖；
> 3. 确认每个链接里都有 `SKILL.md`，告诉我装到了哪里，并提醒我新开一个会话让技能生效；
> 4. 然后按技能 `onboarding` 带我完成注册、MCP token 和 Scout 节点的接入。

用软链接是为了更新只需要 `git pull`，不用重新复制。

### 方式二：手动

```bash
git clone https://github.com/pengchengluoyi/mino.git ~/.mino/skills-src
mkdir -p ~/.claude/skills
for d in ~/.mino/skills-src/skills/*/; do ln -s "${d%/}" ~/.claude/skills/"$(basename "$d")"; done
```

其他 agent 工具把目标目录换成它读取技能的目录。Windows 没有软链接权限时用复制，更新时重新复制。

## 更新

```bash
git -C ~/.mino/skills-src pull --ff-only
```

软链接安装时拉完即最新，新开会话生效。每个版本需要的 Scout / Nexus 版本见 `CHANGELOG.md`。技能有误或缺内容时，见技能 `update-skills`。

## 前提

- 本机已安装并运行 MinoScout（怎么装见 `overview`）；
- agent 已连接 Nexus 的 MCP（`agent_guide` 是总入口）；
- `write-test-cases` 里的方法以本机安装的 `mino_scout/runtime/script_exec.py` 为准。

## 目录规范

- 目录名等于 `SKILL.md` 里的 `name`，小写字母、数字、连字符；
- `SKILL.md` 文件名全大写；
