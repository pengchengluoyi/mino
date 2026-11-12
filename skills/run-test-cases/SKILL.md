---
name: run-test-cases
description: 用本机 mino-scout 确认节点就绪后，通过 Nexus MCP 触发已写好的用例、读任务结果并在对话里汇报。用户要求跑用例、看任务状态、查失败原因时使用。写脚本见 write-test-cases。
---

# 执行用例并汇报

**触发与读结果用 MCP；本机是否能跑先用 `mino-scout` CLI 看。**  
Nexus 管任务 / 设备租约 / 会话；Scout 在用户电脑拉仓执行。测试报告不用另写：`case.nexus.do/check` 已写入会话，对话里只做简要结论 + Workbench 链接。

开跑前用一两句话告诉用户：将跑哪些用例、哪台设备、哪个环境。

## 1. 执行

0. **本机就绪（CLI）**：在 Scout 所在机（通常就是当前机）跑 `mino-scout`（或 `status`）与 `mino-scout probe`。服务没起来、目标平台设备不在 `probe` 里 → 先回 `onboarding` 门禁 3，不要硬 `case_run`。  
1. **应用清单（MCP）**：`repo_apps_get(project_id)`；应用不在清单或 `revision` 落后刚 push 的提交 → `repo-config` §3。无 `repo_url` → `onboarding` 门禁 4 绑仓（`project_script_repo_set`），不要新建无关远程仓。  
2. 选设备：调 `device_list`（带 `app_id`、`env_profile`，用例指定了渠道就带 `env_surface`），拿到**当前账号名下**的设备实时列表，不要复用旧结果。看这几个字段：

   | 字段 | 含义 |
   |---|---|
   | `status` | `online` / `offline` |
   | `executable` | `true` 才是现在能立即用；以它为准 |
   | `busy_task_id` | 非空表示被这个任务占用，空串表示空闲 |
   | `blocking_code` / `blocked_reason` | 不能用的原因，如 `busy`、`node_offline`、`channel_offline`、`app_incompatible`、`capability_missing`、`not_owned`、`unclaimed` |
   | `platform` / `model` / `node_name` | 用来和用例要求的平台匹配、向用户说明 |

   规则：只选 `executable=true` 且平台和用例一致的设备；有多台就列给用户或按用户指定的选；web 设备不会被独占：一台 Scout 同时可以开最多 4 个浏览器会话（4 个并行槽位），所以 `busy_task_id` 始终为空，要看 `web_parallel_full`（4 个都占满才不能再派）；全部 `busy` 就告诉用户谁占着（`busy_task_id`），**不要抢占、不要硬跑**；设备被隔离（`isolation`）时先按 §2 的隔离流程处理。设备是否空闲只由 Nexus 给出，Scout 不会替你判断。
3. **先装齐这次要用的渠道程序，再开跑（MCP）。** `scout_list` 找 `node_id`，`plugin_status(node_id)`。只补本次平台缺的：

   | 这次要跑 | 缺了才装 | 调用 |
   |---|---|---|
   | Web | Chromium | `node_channel_program(node_id, tool_id=chromium, action=install)` |
   | iOS | Appium | `node_channel_program(node_id, tool_id=appium, action=install)` |
   | Android 需要键盘输入 | ADB Keyboard | `node_channel_program(node_id, tool_id=adb.keyboard, action=install)` |

   `operation_id` 自定并记住；装完再用 `plugin_status` 确认。没装完不要 `case_run`。不要安装未列出的插件。
4. `case_run` 传 `script_targets`。`case_list` / `requirement_list` 只返回需求元数据与链接；脚本在 `tests/<需求ID>/<用例ID>.py`。

   ```text
   case_run(
     app_id, sn, revision=<已 push 的 sha>,
     script_targets=[{requirement_id, case_ids}]
   )
   ```

   `case_ids` 为空表示该需求目录下全部脚本。未登记需求先 `requirement_save`。Nexus 不拉仓；Scout 用本机 git 凭据拉。
5. 轮询 `task_get(task_id)`，间隔 ≥5s；终态即停；证据用 `session_events`。

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
