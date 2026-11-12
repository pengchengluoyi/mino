# mino

给 AI agent 用的 Mino 技能（Agent Skills：每个技能一个目录，入口 `SKILL.md`）。

| 技能 | 一句话 |
|---|---|
| `overview` | 分工、MCP vs mino-scout CLI、agent 硬约束 |
| `onboarding` | 注册、接 MCP、`scout_install_token` 确认后装 Scout、建项目绑仓 |
| `repo-config` | 已绑定仓写 `config.yaml` / pages；`check` + `repo_apps_sync` |
| `write-test-cases` | 写用例脚本 |
| `check-test-cases` | `mino-scout check` 自检 |
| `run-test-cases` | CLI 验本机 → MCP 跑数 → 汇报 |
| `update-skills` | 从 GitHub 安装/更新技能（Cursor：**禁止软链**） |

推荐顺序：`overview` → `onboarding` → `repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。

## 安装给 Cursor

从 GitHub **复制**到 `~/.cursor/skills/`（真实目录，不要软链到本机开发仓）。步骤见技能 `update-skills`。

也可对 agent 说：

> 帮我从 https://github.com/pengchengluoyi/mino.git 安装 Mino Skill 到 ~/.cursor/skills（复制，不要软链）。装好后按 onboarding 带我上手。

装完后**新开一个会话**。版本见已装目录里的 `VERSION`、`CHANGELOG.md`。

## 更新

再跑一遍 `update-skills` 里的安装命令（重新 clone + 覆盖复制）。不要 `ln -s` 到 `code/mino`。

## 前提

- Scout：按 `onboarding` 用 MCP 申请安装命令并确认后安装，CLI 验收；  
- MCP：`https://nexus.minocc.top/mcp`（先 `agent_guide`）；  
- Workbench：`https://workbench.minocc.top`；  
- 写作用法以本机 `mino_scout/runtime/script_exec.py` 为准。

## 目录规范

- 目录名等于 `SKILL.md` 的 `name`；  
- `SKILL.md` 文件名全大写。
