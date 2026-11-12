---
name: run-test-cases
description: 通过 Nexus 的 MCP 工具执行已写好的用例、读取任务和会话结果，并在对话里向用户汇报。用户要求跑用例、看任务状态、查失败原因时使用。写脚本见 write-test-cases。
---

# 执行用例并汇报（Nexus）

Nexus 管用例、设备、账号、任务、会话；Scout 在用户电脑上执行。你通过 Nexus 的 MCP 工具触发执行、读结果。

**测试报告不用你另写一份**：脚本里 `case.nexus.do/check(title)` 已经把每一步的成败和证据写进会话，Workbench 展示任务、会话、截图。你在对话里只做一件事：向用户简要说明结果，并给 Workbench 链接。

## 1. 执行

0. 确认应用：`repo_apps_get(project_id)` 看这个项目的仓库能跑哪些应用和环境；要跑的应用不在里面，或同步的 `revision` 落后于你刚 push 的提交，先回到 `repo-config` §3 同步，再继续。项目还没有脚本仓库地址时，用 `project_script_repo_set(project_id, url, branch)` 保存地址（Nexus 只记地址，不拉仓库）；
1. 选设备：调 `device_list`（带 `app_id`、`env_profile`，用例指定了渠道就带 `env_surface`），拿到**当前账号名下**的设备实时列表，不要复用旧结果。看这几个字段：

   | 字段 | 含义 |
   |---|---|
   | `status` | `online` / `offline` |
   | `executable` | `true` 才是现在能立即用；以它为准 |
   | `busy_task_id` | 非空表示被这个任务占用，空串表示空闲 |
   | `blocking_code` / `blocked_reason` | 不能用的原因，如 `busy`、`node_offline`、`channel_offline`、`app_incompatible`、`capability_missing`、`not_owned`、`unclaimed` |
   | `platform` / `model` / `node_name` | 用来和用例要求的平台匹配、向用户说明 |

   规则：只选 `executable=true` 且平台和用例一致的设备；有多台就列给用户或按用户指定的选；web 设备不会被独占：一台 Scout 同时可以开最多 4 个浏览器会话（4 个并行槽位），所以 `busy_task_id` 始终为空，要看 `web_parallel_full`（4 个都占满才不能再派）；全部 `busy` 就告诉用户谁占着（`busy_task_id`），**不要抢占、不要硬跑**；设备被隔离（`isolation`）时先按 §2 的隔离流程处理。设备是否空闲只由 Nexus 给出，Scout 不会替你判断。
2. **先装齐这次要用的渠道程序，再开跑。** 用 `scout_list` 找到持有这台设备的 `node_id`，调 `plugin_status(node_id)`。只补这次平台缺的组件，已经装好的不要重装：

   | 这次要跑 | 缺了才装 | 调用 |
   |---|---|---|
   | Web | Chromium | `node_channel_program(node_id, tool_id=chromium, action=install)` |
   | iOS | Appium | `node_channel_program(node_id, tool_id=appium, action=install)` |
   | Android 需要键盘输入 | ADB Keyboard | `node_channel_program(node_id, tool_id=adb.keyboard, action=install)` |

   `operation_id` 自己定一个并记住。返回后隔几秒再 `plugin_status(node_id)`，直到这次安装不再进行中。没装完不要 `case_run`。重试必须沿用同一个 `operation_id`。看到 `declined / 缺 appium`（或其它渠道程序）说明顺序反了：先装完，再跑，不要先跑再补装。
3. `case_run` 传 `script_targets`，不要拿用例库里的正文。`case_list` / `requirement_list` 只返回需求 id、名称和渠道链接。从链接读出用例后，脚本在 `tests/<需求ID>/<用例ID>.py`。

   ```text
   case_run(
     app_id, sn, revision=<已 push 的 sha>,
     script_targets=[{requirement_id, case_ids}]
   )
   ```

   `case_ids` 是这次要跑的用例 ID。留空表示执行该需求目录下的全部脚本。需求还没登记时先 `requirement_save(project_id, requirement_id, title, source_url)`。顶层 `case_ids` 只在你已经知道脚本文件名、且不按需求目录跑时使用。`revision` 可省略（Scout 取分支最新）。Nexus 不经手脚本仓库，由设备所在的 Scout 用本机 git 凭据拉取。
4. 轮询 `task_get(task_id)`，两次查询间隔 ≥5s；用工具调用本身的等待，不要阻塞式 `sleep`；
5. 终态即停止；过程证据读 `session_events(session_id)`。

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
| `otp_requires_human` | 账号无自带码且 `config.yaml` 未写 `otp.code`；向用户要本次验证码，或写入测试用 `otp.code` 后 push 再跑 |
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
