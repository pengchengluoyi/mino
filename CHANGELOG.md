# 更新记录

每个版本写清需要的 Scout / Nexus 版本。技能说的和实际对不上时，先核对这里。

## 0.1.10

- **约束 agent**：`overview` 硬约束（先告知再动手、禁止擅自建远程仓、不把 `app_create` 当完成）。
- **MCP vs CLI**：本机 `mino-scout`（status / probe / check），云端 MCP。
- **`onboarding` 门禁**：注册 → MCP → **`scout_install_token` 申请安装命令 → 确认弹窗 → 本机安装**（禁止 Workbench 抄命令 / 用户粘贴）→ `project_create` / 绑仓 / sync（直接调 MCP，不写「没工具就去网页」绕路）→ 需求与用例关系说明；示例 key 用 `baidu_*`。
- **`repo-config` / `write-test-cases` / `run-test-cases` / `check-test-cases`**：对齐绑仓、CLI 探活、需求列表不含用例正文。

需要线上 MCP 含：`project_create`、`scout_install_token`（`POST /runtime/nodes/install-token`）、`project_script_repo_set`、`repo_apps_sync` / `get`、`app_delete` 等（与 Cloudflare README / HTTP 对齐）。

## 0.1.9

- `run-test-cases`：开跑前先 `plugin_status`，缺的渠道程序按需装完再 `case_run`。执行用 `script_targets`（需求 ID + 用例 ID），不从用例库取正文。
- `onboarding`：脚本仓库用 `project_script_repo_set`；需求用 `requirement_save`，`case_list` 返回需求链接。
- `update-skills`：更新改为在 mino 检出里 `git pull`，不把技能文件复制到本地技能目录。

需要已部署的 Cloudflare Nexus（含 `script_targets`、`requirement_list`、`project_script_repo_set`、`repo_apps_sync`）和 Scout 0.1.143（`script.list`）。

## 0.1.8

- `repo-config`：新增 §0「新增应用：先问齐再写」——加安卓 / iOS / Web 前必须问清项目环境与各环境 package / bundle_id / url；完成态是写 `config.yaml` + push + `repo_apps_sync`，**仅 `app_create` 不算完成**。
- `onboarding` 第四节补「先确认环境再加应用」；`overview` 路由表增加「加应用 / 配包名」指向 `repo-config`。

需要 Scout / Nexus 版本同 0.1.7（Scout 0.1.141、Nexus 0.1.80）。

## 0.1.7

- 脚本仓优先：`config.yaml` 只认应用 key，**不再要求 `app_id` / `project_id`**；`repo_apps_sync` 可不传 `app_id`，Nexus 按 key 对齐或自动创建应用。
- MCP 边界写清：只通信（查询、同步元数据、触发跑数），不写脚本；脚本由 agent 本机编写并 Git 推送。
- 固定验证码迁到脚本仓：可选 `otp.code`；不写则 `case.scout.otp` 报 `otp_requires_human`，向人要码。同步载荷禁止带 `otp`。
- 登录动作仍在 `flows/`；账号仍由 Nexus 号池租用。

需要 Scout 0.1.141、Nexus 0.1.80。

## 0.1.6

- 技能安装：把仓库地址交给 agent，按该 agent 惯用方式安装（技能在 `skills/`）。
- `overview`：说明 Nexus / Workbench / Scout 三端分工；用户电脑只装 Scout，本地状态在 `~/.mino/`。

需要已部署的 Cloudflare Nexus（`nexus.minocc.top`）与 Workbench Pages。

## 0.1.5

- `onboarding` / `overview` / `README`：默认使用线上 MCP 与 Workbench 地址。

需要已部署的 Cloudflare Nexus（`nexus.minocc.top`）与 Workbench Pages。

## 0.1.4

- 写用例先选定一台可执行设备。页面定位、点击、截图和界面结构都写在用例脚本里，由 `case_run` 交给 Scout 的 `script.run`。
- `locator_spec` 只从这次运行 `tool/result` 里 `capability_id=hierarchy` 的 `nodes` 或 `hierarchy_text` 抄。写完覆盖原文的脚本后，用同一台设备再跑一次调试。

需要 Scout 0.1.136、Nexus 0.1.78 把 `app.hierarchy()` 的 `nodes` / `hierarchy_text` 写入该次 `tool/result`。

## 0.1.3

- `run-test-cases` 只写通过 `case_run` 跑脚本仓。
- `overview` 里 Nexus 的职责与现在的派发方式一致：下发仓库、提交和设备，脚本在 Scout 执行。
- `onboarding`、`overview`、`README`：MCP 接入是浏览器里的 OAuth。配置只写 `<NEXUS>/mcp`。

需要 Scout 0.1.134、Nexus 0.1.76。

## 0.1.2

- `write-test-cases`：前置写成 `prep`，操作和预期至少各覆盖原文每一条，一条可以拆成 `2-1`、`2-2`；清理缓存用 `app.clear_cache()`。没有动作要做时也写一条 `prep`，标题说明为什么。
- `locator_spec` 只抄设备界面节点上的 `resource_id`、`text` 或 `content_desc`。没有节点就不写选择器，也不写对应的 `tap` / `input`。
- `check-test-cases`：`tests/` 下的用例必须有 `prep`，标题只写「无」不算。自检不证明选择器来自真机。
- `run-test-cases`：通过 `case_run` 跑项目脚本仓。

需要 Scout 0.1.134、Nexus 0.1.76。`prep` 的标题沿用现有步骤回调写入 Nexus。

## 0.1.1

首版，七个技能：`overview`、`onboarding`、`repo-config`、`write-test-cases`、`check-test-cases`、`run-test-cases`、`update-skills`。

需要的 Scout 和 Nexus 版本及变更：
- Scout 0.1.133 起、Nexus 0.1.75 起；
- `script.run` 改为下发 `repo_url` / `branch` / `revision`，Scout 自己拉仓库；
- `case.init(case.config.<KEY>)` 的 `app` / `case.scout` / `case.nexus` 写法，`case.nexus.do("标题")`；
- `mino-scout check`；
- MCP 工具 `repo_apps_sync`（差异更新）、`repo_apps_get`、`app_create`；`config.yaml` 每个应用带 `app_id`。
