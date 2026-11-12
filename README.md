# mino

给 AI agent 用的 Mino 技能（Agent Skills：每个技能一个目录，入口 `SKILL.md`）。

| 技能 | 一句话 |
|---|---|
| `overview` | 分工、**MCP vs mino-scout CLI**、agent 硬约束 |
| `onboarding` | 注册、接 MCP、`scout_install_token` 确认后装 Scout、`project_create`→绑仓→应用 |
| `repo-config` | 在已绑定仓写 `config.yaml` / pages；`check` + `repo_apps_sync`；禁止擅自建远程仓 |
| `write-test-cases` | 写用例脚本（`app` / `case.scout` / `case.nexus`） |
| `check-test-cases` | `mino-scout check` 自检 |
| `run-test-cases` | CLI 验本机 → MCP 触发跑数 → 对话汇报 |
| `update-skills` | 更新技能与反馈 |

推荐顺序：`overview` → `onboarding` → `repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。

## 安装给 agent

把下面发给你正在用的 agent，按它惯用方式安装。技能在仓库 `skills/`。

> 帮我安装这个 Skill：https://github.com/pengchengluoyi/mino.git  
> 装好后按技能 `onboarding` 带我完成注册、OAuth 接 MCP、安装 Scout，再配置项目与脚本仓。

装完后**新开一个会话**再生效。版本见根目录 `VERSION`、`CHANGELOG.md`。

## 更新

见 `update-skills`：在 mino 检出里 `git pull`，不要复制技能文件到 agent 目录。

## 前提

- 本机将安装并运行 MinoScout（流程在 `onboarding`，用 `mino-scout` CLI 验收）；
- agent 已接 Nexus MCP：`https://nexus.minocc.top/mcp`（先调 `agent_guide`）；
- Workbench：`https://workbench.minocc.top`（注册、OAuth、生成 Scout 安装命令）；
- `write-test-cases` 方法以本机 `mino_scout/runtime/script_exec.py` 为准。

## 目录规范

- 目录名等于 `SKILL.md` 的 `name`；
- `SKILL.md` 文件名全大写。
