---
name: run-test-cases
description: 通过 Nexus 的 MCP 工具执行已写好的用例、读取任务和会话结果，并在对话里向用户汇报。用户要求跑用例、看任务状态、查失败原因时使用。写脚本见 write-test-cases。
---

# 执行用例并汇报（Nexus）

Nexus 管用例、设备、账号、任务、会话；Scout 在用户电脑上执行。你通过 Nexus 的 MCP 工具触发执行、读结果。

**测试报告不用你另写一份**：脚本里 `case.nexus.do/check(title)` 已经把每一步的成败和证据写进会话，Workbench 展示任务、会话、截图。你在对话里只做一件事：向用户简要说明结果，并给 Workbench 链接。

## 1. 执行

0. 确认应用：`repo_apps_get(project_id)` 看这个项目的仓库能跑哪些应用和环境；要跑的应用不在里面，或同步的 `revision` 落后于你刚 push 的提交，先回到 `repo-config` §3 同步，再继续；
1. 选设备：调 `device_list`（带 `app_id`、`env_profile`，用例指定了渠道就带 `env_surface`），拿到**当前账号名下**的设备实时列表，不要复用旧结果。看这几个字段：

   | 字段 | 含义 |
   |---|---|
   | `status` | `online` / `offline` |
   | `executable` | `true` 才是现在能立即用；以它为准 |
   | `busy_task_id` | 非空表示被这个任务占用，空串表示空闲 |
   | `blocking_code` / `blocked_reason` | 不能用的原因，如 `busy`、`node_offline`、`channel_offline`、`app_incompatible`、`capability_missing`、`not_owned`、`unclaimed` |
   | `platform` / `model` / `node_name` | 用来和用例要求的平台匹配、向用户说明 |

   规则：只选 `executable=true` 且平台和用例一致的设备；有多台就列给用户或按用户指定的选；web 设备不会被独占：一台 Scout 同时可以开最多 4 个浏览器会话（4 个并行槽位），所以 `busy_task_id` 始终为空，要看 `web_parallel_full`（4 个都占满才不能再派）；全部 `busy` 就告诉用户谁占着（`busy_task_id`），**不要抢占、不要硬跑**；设备被隔离（`isolation`）时先按 §2 的隔离流程处理。设备是否空闲只由 Nexus 给出，Scout 不会替你判断。
2. `case_run(app_id, sn, case_ids, revision=<sha>)` → 立即返回 `task_id`（异步）。脚本在项目脚本仓里，由 Scout 拉取该提交后执行。`revision` 是已 push 的 commit，可省略（省略时 Scout 取分支最新）；Nexus 不经手脚本仓库，由设备所在的 Scout 用它本机的 git 凭据自己拉取；
3. 轮询 `task_get(task_id)`，两次查询间隔 ≥5s；用工具调用本身的等待，不要阻塞式 `sleep`；
4. 终态即停止；过程证据读 `session_events(session_id)`。

| `task_get.status` | 动作 |
|---|---|
| `queued` | 等 5s 再查；不重复 `case_run` |
| `running` | 可读事件；不打断、不改脚本 |
| `pass` | 汇报通过 |
| `fail` | 先看 `cases[].status` 和 `reason_code`，再归因 |
| `blocked` | 看 `resource_isolated`；需要时 baseline |
| `cancelled` | 报告取消；不擅自重跑 |

任务级只有这几种状态。Scout 的 `declined` / `skipped` 在任务级聚合为 `fail`，区分在 `cases[].status`：

| `cases[].status` | 怎么说 |
|---|---|
| `declined` | “Scout 拒绝执行”，附 `reason_code`；不说环境故障 |
| `skipped` | “未执行”，说明前置条件 |
| `fail` | 失败，给失败的 do/check 标题 |
| `blocked` | 阻塞，说明等人还是等资源 |

禁止：并发对同一 `case_id + sn` 重复 `case_run`；看到 `fail` 就自动重跑。

## 2. 读结果

- `task_get`：`status`、`cases[]`、`pass_rate`、`resource_isolated`；
- `session_events`：字段含义以实际返回为准。`tool/call` 与 `tool/result` 成对，`tool/result` 的 `app` 标明是哪个应用的动作，`node/enter`、`node/leave` 对应脚本里的 do/check 标题。`capability_id=hierarchy` 的 `nodes`（没有则 `hierarchy_text`）是这次 Scout 读到的界面结构，写页面定位只用它；截图用 `session_screenshot`。

需要特别处理的 `reason_code`：

| `reason_code` | 怎么办 |
|---|---|
| `locator_missing` / `control_unknown` | 改 `pages/<APP_KEY>/*.yaml`；脚本问题，不是产品缺陷 |
| `method_unknown` | 方法名写错，报错里有最相近的名字；改脚本 |
| `config_missing` | 脚本仓库的 `config.yaml` 缺这个应用或这个环境，见 `repo-config` |
| `app_not_in_repo_config` | Nexus 里没有同步这个应用，见 `repo-config` §3 |
| `vlm_failed` | 视觉判定未通过，看 `script/note` 和截图，再判断是产品问题还是判定词问题 |
| `revision_missing` | Scout 在仓库里取不到这个提交：确认已 push；不要回退到分支最新 |
| `repo_unavailable` | 设备所在电脑拉不到仓库：本机没有 git，或没有该仓库的 git 凭据；让用户处理那台电脑 |
| `script_missing` / `script_ambiguous` | 仓库里找不到或有多份 `tests/**/<case_id>.py` |
| `write_outcome_unknown` | 保持隔离；baseline 只释放资源；汇报 blocked |

其余按返回的 `summary` 原样转述。脚本问题改完要回到 `write-test-cases` 和 `check-test-cases`，重新 push 再跑。

### 隔离

`resource_isolated=true` 时调用 `task_write_safety_baseline(task_id)`，它补拍并释放设备 / 账号租约；原任务结论不变，汇报里同时写“已释放”和“结论仍为 blocked”；截图失败就继续隔离，不手工释放。

## 3. 在对话里怎么汇报

简短，四件事：**结论**（几条通过）、**失败的是哪个 do/check 标题和原因**、**归因**（产品缺陷 / 脚本问题 / 环境问题）、**Workbench 链接**。有环境信息就带上平台、设备、App 版本、revision。

规则：

- 没有 `session_events` 证据，不下结论；
- 不把 `blocked` 说成 `failed`，不把 `declined` 说成环境故障；
- 归因是你的判断，说明依据，不确定就说不确定。
