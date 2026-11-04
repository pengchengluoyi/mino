# 更新记录

每个版本写清需要的 Scout / Nexus 版本。技能说的和实际对不上时，先核对这里。

## 0.1.1

首版，七个技能：`overview`、`onboarding`、`repo-config`、`write-test-cases`、`check-test-cases`、`run-test-cases`、`update-skills`。

需要的 Scout 和 Nexus 版本及变更：
- Scout 0.1.133 起、Nexus 0.1.75 起；
- `script.run` 改为下发 `repo_url` / `branch` / `revision`，Scout 自己拉仓库；
- `case.init(case.config.<KEY>)` 的 `app` / `case.scout` / `case.nexus` 写法，`case.nexus.do("标题")`；
- `mino-scout check`；
- MCP 工具 `repo_apps_sync`（差异更新）、`repo_apps_get`、`app_create`；`config.yaml` 每个应用带 `app_id`。
