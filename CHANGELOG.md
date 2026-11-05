# 更新记录

每个版本写清需要的 Scout / Nexus 版本。技能说的和实际对不上时，先核对这里。

## 0.1.4

- 写用例先选定一台可执行设备。页面定位、点击、截图和界面结构都写在用例脚本里，由 `case_run` 交给 Scout 的 `script.run`。
- `locator_spec` 只从这次运行 `tool/result` 里 `capability_id=hierarchy` 的 `nodes` 或 `hierarchy_text` 抄。写完覆盖原文的脚本后，用同一台设备再跑一次调试。

需要 Scout 0.1.136、Nexus 0.1.78 把 `app.hierarchy()` 的 `nodes` / `hierarchy_text` 写入该次 `tool/result`。

## 0.1.3

- `run-test-cases` 只写通过 `case_run` 跑脚本仓。
- `overview` 里 Nexus 的职责与现在的派发方式一致：下发仓库、提交和设备，脚本在 Scout 执行。
- `onboarding`、`overview`、`README`：MCP 接入是浏览器里的 OAuth。配置只写 `<NEXUS>/mcp`。
- 对话读取 `~/.claude/skills/` 里的技能目录。更新时从仓库复制进去。

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
