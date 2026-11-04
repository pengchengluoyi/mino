---
name: repo-config
description: 脚本仓库怎么组织、config.yaml 怎么配置被测应用和不同环境的地址、页面文件 pages 怎么写。新建或修改脚本仓库的目录、应用、环境、页面时使用；写具体用例见 write-test-cases。
---

# 脚本仓库配置

脚本仓库是项目的 Git 仓库（地址和分支在 Nexus 的 `project_list` 里）。执行时 **Scout 自己用本机 git 拉取这个仓库**（私有仓库用 Scout 所在电脑的 git 凭据），Nexus 只告诉它仓库地址、分支、提交和设备，不经手仓库内容。

## 1. 目录构成

| 路径 | 放什么 | 判断标准 |
|---|---|---|
| `config.yaml` | 被测应用清单：key、平台、各环境的包名 / bundle id / URL | 只放应用身份，不放账号密码 |
| `pages/<APP_KEY>/<page_key>.yaml` | 某个应用的页面：状态锚点、控件、各渠道 locator | 只描述“页面是什么”，不含动作；一个应用一个子目录 |
| `tests/<需求ID>/<case_id>.py` | 用例，`def run(case)` | 一条用例一个文件；不写平台后缀 |
| `flows/<name>.py` | 业务流程：登录、进入某页、下单；多个用例复用 | 函数接收 `app`，编排设备动作 |
| `scripts/<name>.py` | 纯逻辑：数据处理、格式转换、计算 | 入参出参是普通数据，不碰设备 |
| `pretest.py` / `posttest.py` | 项目级前后置钩子 | 可选 |

写之前先找现成的，**不要重复造轮子**：Scout / Nexus 提供的方法 → `pages` 里的控件 → 已有 `flows` → 新写 `flows` → 纯逻辑才写 `scripts`。引用写 `from flows.login import login`。

## 2. config.yaml

放仓库根目录。顶层只有 `apps`，每个应用一项。**key 用英文字母开头的英文、数字、下划线**（`camera`、`shop_web`、`APP01` 都行，纯英文也可以），这个 key 同时是脚本里 `case.config.<KEY>` 的名字（所以要是合法的 Python 变量名）和 `pages/<KEY>/` 的目录名。

每个应用**必须有 `app_id`**：它是这个应用在 Nexus 里的 id（形如 `APP001`），用 MCP `project_list` 查项目下的应用拿到。执行时 Nexus 用它决定跑哪个应用。应用在 Nexus 里还不存在，先调 MCP `app_create(project_id, name, platforms)` 创建并拿到 id（创建前先用 `project_list` 确认同名应用不存在，不要重复创建），**不要自己编 id**。

应用在不同环境的身份不同（测试包名、测试域名）时用 `envs` 分环境；环境名与 Nexus 项目里的环境名一致（如 `test`、`prod`）：

```yaml
apps:
  camera:
    app_id: app_8f3a21        # Nexus 里这个应用的 id
    platform: android
    envs:
      test: {package: com.example.a.test}
      prod: {package: com.example.a}
  shop_web:
    app_id: app_c71d09
    platform: web
    envs:
      test: {url: https://test.example.com}
      prod: {url: https://example.com}
  camera_ios:                # 各环境身份相同，可以不写 envs
    app_id: app_8f3a21
    platform: ios
    bundle_id: com.example.a
```

各平台身份字段：

| platform | 字段 | 说明 |
|---|---|---|
| `android` | `package` | 包名 |
| `ios` | `bundle_id` | bundle id |
| `web` | `url` | `launch()` 打开的地址 |
| `mac` | `bundle_id` | bundle id |
| `win` | `package` | EXE 路径或 AUMID |

规则：

- 值从用户给的信息、用例前置条件或已有 `config.yaml` 里取；**拿不到包名 / URL / 环境就问用户，不要猜**；
- 已有应用的 key 不要改，改了会让 `pages/` 目录和已有用例对不上；
- 只放应用身份，不放账号、密码、验证码、token（账号由 Nexus 号池租用）；
- 新增应用 = 在 Nexus 有这个应用（拿到 `app_id`）+ `config.yaml` 加一项 + 建 `pages/<KEY>/` 目录；
- 应用的 `platform` 必须等于本次运行的设备平台；
- 本次运行的环境由 Nexus 决定，脚本里是 `case.env`，一般不需要判断；
- 这是应用地址的唯一真源，由 Scout 读取；Workbench 里的包名 / URL 只读展示，要改就改这个文件并 push，再按下面的约定同步给 Nexus。

## 3. 改了 config.yaml 必须同步给 Nexus

Nexus 不读脚本仓库（仓库可能是私有的），它只能靠你告诉它“这个项目的仓库能跑哪些应用”，Workbench 的“脚本仓库应用”和选应用都依赖这份同步。所以**只要新增、删除或修改了 `config.yaml` 里的应用或环境**，在 push 之后：

1. 取 push 后的提交：`git rev-parse HEAD`；
2. 调 MCP 工具 `repo_apps_sync(project_id, revision=<sha>, branch=<分支>, apps=<本次新增或修改的应用>, remove=<本次删除的应用 key>)`。这是差异更新：只传变化的应用（取自 `config.yaml`，按 key 整项替换），没传的应用保持不变；删除应用要显式放进 `remove`；
3. 调 `repo_apps_get(project_id)` 确认同步成功，`revision` 与刚 push 的一致。

规则：

- 先 push 再同步，同步的 `revision` 必须是已 push 的提交；
- 传的应用原样取自 `config.yaml`（每项含 `app_id`），不要自己增删字段，不要把账号、密码、token 放进去（含这类键名会被拒绝）；
- 同步被拒绝（key 命名、platform、字段不合法）就按报错改 `config.yaml`，改好重新 push 再同步；
- 只改了脚本、没动 `config.yaml`，不需要同步；
- 新项目第一次写好 `config.yaml` 后也要同步一次。

## 4. 页面文件

`pages/<APP_KEY>/<page_key>.yaml`，目录名必须等于 `config.yaml` 里的 key；直接放在 `pages/` 下不行。字段参照仓库里已有的页面文件：`page_key`、`states[].anchors`、`controls[].id` / `label` / `locator_spec`。

`locator_spec` 按渠道分格：

| 渠道 | 格名 |
|---|---|
| android | `uiautomator` |
| ios | `xcuitest` |
| web | `playwright` |
| mac | `ax` |
| win | `uia` |

每格 `selector` 里有 `kind` 和 `value` / `name`。`on_page` 用 `states[].anchors` 与界面文字比对，**一组锚点全部出现**才算在这一页。同一控件要在多个渠道用，就在 `locator_spec` 里写多格，不要在脚本里按平台分支。
