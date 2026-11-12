---
name: overview
description: 了解 Mino 是什么、Scout / Nexus / Workbench / Agent 各管什么、MCP 与 mino-scout CLI 怎么分工。用户问“系统怎么分工”“Scout 是什么”“该用 MCP 还是 CLI”，或你不确定该找哪一端时使用。
---

# Mino 总览

Mino 是自动化测试系统：AI 写脚本，真机 / 浏览器执行并留证据。四端各管一件事，**互相不越界**。

| 端 | 是什么 | 负责 | 不负责 |
|---|---|---|---|
| **Nexus** | 云端（`https://nexus.minocc.top`） | 项目 / 应用 / 需求元数据、设备与账号、任务派发、事件存储；对 Workbench 与 agent（MCP）供数 | 写脚本；拉 Git；操作设备；存 Git 令牌 |
| **Scout** | 本机执行器（`mino-scout`） | 连 Nexus、拉脚本仓、跑脚本、操作设备、截图取证、本机取码 / 调模型 | 选用例；决定“下一步测什么” |
| **Workbench** | 网页（`https://workbench.minocc.top`） | 给人看的管理与证据；注册 / OAuth | 写脚本；替 agent 申请安装命令或建项目 |
| **Agent** | 对话侧 | 按技能问清 → MCP 装 Scout / 建项目绑仓 → 写仓 → CLI 自检 → 触发跑数；每步告知用户 | 擅自建远程仓、猜包名 / URL、让用户手抄安装命令 |

本地状态在 `~/.mino/`。Scout 主动连 Nexus，不开放监听端口。

## MCP 与 mino-scout CLI（必分清）

| 事 | 用什么 | 例子 |
|---|---|---|
| 查 / 改 Nexus 控制面 | **MCP** | `project_list`、`project_create`、`project_script_repo_set`、`repo_apps_sync` / `get`、`scout_install_token`、`device_list`、`case_run`、`task_get`、渠道程序安装 |
| 本机 Scout | **`mino-scout` CLI** | 服务状态、`probe` 设备、仓根 `check` 自检（以本机 `--help` 为准） |
| 写文件与推送 | **本机 Git** | 编辑脚本仓 → commit → push；MCP **不写**仓 |

技能正文若只堆 MCP、不提 CLI，是错的：本机就绪与脚本质量靠 Scout CLI；云端编排靠 MCP。

## Agent 硬约束（所有技能共用）

1. **先读技能再动手**；缺信息就问，**不要猜**包名、URL、仓库地址、环境名。
2. **每进入一个大步骤，先用一两句话告诉用户**：将做什么、需要用户确认什么、成功长什么样；做完再报结果。不要闷头连调工具。
3. **禁止**在用户未明确说「新建仓库 / 创建 GitHub 仓」时 `gh repo create`、初始化并推到新远程。脚本仓地址必须用户给出，或明确授权新建。
4. **禁止**把「只调了 `app_create`」说成应用已配好。完成态见 `onboarding` / `repo-config`。
5. MCP 工具缺失或报错：**停住**，告诉用户缺哪个工具、卡在哪一步；不要改去刷 Workbench、编造 HTTP、或用探测应用「试接口」。
6. 会改系统的动作（装 Scout、装渠道程序、push、`case_run`）前要有用户意图；装 Scout 还必须用户同意执行安装命令。
7. 凭据、安装命令里的一次性 token：**不写入回复正文、日志、记忆、提交**。

## 一次测试的流向

1. 项目 + 脚本仓绑定 + 应用清单就绪（`onboarding` / `repo-config`）；
2. agent 在本机写脚本并 `mino-scout check`，再 push；
3. 改了 `config.yaml` 应用身份 → `repo_apps_sync`；
4. MCP `case_run` → Nexus 把仓地址 / 提交 / 设备下给 Scout；
5. Scout 本机拉仓执行，证据回 Nexus；对话里简要汇报。

## 该用哪个技能

| 你要做的事 | 技能 |
|---|---|
| 分工、MCP vs CLI、硬约束 | 本技能 |
| 注册、接 MCP、**MCP 装 Scout**、`project_create` / 绑仓 / sync | `onboarding` |
| 改 `config.yaml` / pages、加应用（问齐再写） | `repo-config` |
| 写用例脚本 | `write-test-cases` |
| `mino-scout check` | `check-test-cases` |
| 触发跑数、看任务、汇报 | `run-test-cases` |
| 更新技能 | `update-skills` |

Scout 安装：MCP `scout_install_token` 取命令 → 用户确认弹窗同意 → 本机执行；细节只在 `onboarding` 门禁 3。
