# mino

给 AI agent 用的 Mino 技能（Agent Skills 标准：每个技能一个目录，入口是 `SKILL.md`）。

| 技能 | 一句话 |
|---|---|
| `overview` | Mino 是什么，Scout / Nexus / Workbench 三端各管什么，Scout 怎么装 |
| `onboarding` | 第一次使用：注册登录、用 OAuth 接入 MCP、装 Scout 节点、准备项目与脚本仓库 |
| `repo-config` | 脚本仓库怎么组织，`config.yaml` 怎么配应用 / 环境 / 可选 `otp.code`，改了怎么同步给 Nexus（按 key，不写 `app_id`） |
| `write-test-cases` | 为用例写脚本（Scout 侧）：`app` / `case.scout` / `case.nexus` 怎么用 |
| `check-test-cases` | 写完用 `mino-scout check` 自检，按提示改 |
| `run-test-cases` | 通过 Nexus 执行用例、选设备、看结果、在对话里汇报 |
| `update-skills` | 更新技能；技能和实际对不上时怎么反馈 |

每个技能只做一件事。推荐顺序：`overview` → `onboarding` → `repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。

## 安装给 agent

把下面这段发给你正在用的 agent，让它按自己惯用的方式安装。技能在仓库的 `skills/` 下。

> 帮我安装这个 Skill：https://github.com/pengchengluoyi/mino.git
> 装好后按技能 `onboarding` 带我完成注册、OAuth 接入 MCP 和 Scout 节点。

装完后**新开一个会话**再让技能生效。版本与变更见仓库根目录的 `VERSION`、`CHANGELOG.md`。

## 更新

同样让 agent 从同一地址按它自己的方式更新，见技能 `update-skills`。

## 前提

- 本机已安装并运行 MinoScout（怎么装见 `overview`）；
- agent 已连接 Nexus MCP：`https://nexus.minocc.top/mcp`（`agent_guide` 是总入口）；
- Workbench：`https://workbench.minocc.top`；
- `write-test-cases` 里的方法以本机安装的 `mino_scout/runtime/script_exec.py` 为准。

## 目录规范

- 目录名等于 `SKILL.md` 里的 `name`，小写字母、数字、连字符；
- `SKILL.md` 文件名全大写；
