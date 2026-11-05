# 更新记录

每个版本写清需要的 Scout / Nexus 版本。技能说的和实际对不上时，先核对这里。

## 0.1.2

- `write-test-cases`：前置写成 `prep`，操作和预期至少各覆盖原文每一条，一条可以拆成 `2-1`、`2-2`；清理缓存用 `app.clear_cache()`。没有动作要做时也写一条 `prep`，标题说明为什么。
- `locator_spec` 只抄设备界面节点上的 `resource_id`、`text` 或 `content_desc`。没有节点就不写选择器，也不写对应的 `tap` / `input`。
- `check-test-cases`：`tests/` 下的用例必须有 `prep`，标题只写「无」不算。自检不证明选择器来自真机。
- `run-test-cases`：只通过 `case_run` 跑脚本仓。不传 `action_scheme`，没有 copilot，没有外部自行执行。

需要 Scout 0.1.134、Nexus 0.1.76。`prep` 的标题沿用现有步骤回调写入 Nexus。

## 0.1.1

首版，七个技能：`overview`、`onboarding`、`repo-config`、`write-test-cases`、`check-test-cases`、`run-test-cases`、`update-skills`。

需要的 Scout 和 Nexus 版本及变更：
- Scout 0.1.133 起、Nexus 0.1.75 起；
- `script.run` 改为下发 `repo_url` / `branch` / `revision`，Scout 自己拉仓库；
- `case.init(case.config.<KEY>)` 的 `app` / `case.scout` / `case.nexus` 写法，`case.nexus.do("标题")`；
- `mino-scout check`；
- MCP 工具 `repo_apps_sync`（差异更新）、`repo_apps_get`、`app_create`；`config.yaml` 每个应用带 `app_id`。
