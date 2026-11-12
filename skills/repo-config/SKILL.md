---
name: repo-config
description: 脚本仓库怎么组织、config.yaml 怎么配置被测应用和不同环境的地址与可选固定验证码、页面文件 pages 怎么写。新建或修改脚本仓库的目录、应用、环境、页面时使用；写具体用例见 write-test-cases。
---

# 脚本仓库配置

脚本仓库是项目的 Git 仓库（地址和分支在 Nexus 的 `project_list` 里）。执行时 **Scout 自己用本机 git 拉取这个仓库**（私有仓库用 Scout 所在电脑的 git 凭据），Nexus 只告诉它仓库地址、分支、提交和设备，不经手仓库内容。

**边界：** 脚本文本只由 agent 在本机写并通过 Git 推送；MCP 只通信（查项目、同步应用清单、触发跑数），不写代码、不改仓库文件。脚本仓认应用 **key**；`project_id` / `app_id` 是 Nexus 控制面身份，**不要写进 `config.yaml`**。

## 1. 目录构成

| 路径 | 放什么 | 判断标准 |
|---|---|---|
| `config.yaml` | 被测应用清单：key、平台、各环境的包名 / bundle id / URL、可选 `otp.code` | 只放应用身份与取码声明，不放账号密码 |
| `pages/<APP_KEY>/<page_key>.yaml` | 某个应用的页面：状态锚点、控件、各渠道 locator | 只描述“页面是什么”，不含动作；一个应用一个子目录 |
| `tests/<需求ID>/<case_id>.py` | 用例，`def run(case)` | 一条用例一个文件；不写平台后缀 |
| `flows/<name>.py` | 业务流程：登录、进入某页、下单；多个用例复用 | 函数接收 `app`，编排设备动作 |
| `scripts/<name>.py` | 纯逻辑：数据处理、格式转换、计算 | 入参出参是普通数据，不碰设备 |
| `pretest.py` / `posttest.py` | 项目级前后置钩子 | 可选 |

写之前先找现成的，**不要重复造轮子**：Scout / Nexus 提供的方法 → `pages` 里的控件 → 已有 `flows` → 新写 `flows` → 纯逻辑才写 `scripts`。引用写 `from flows.login import login`。登录**动作**写在 `flows/`；账号仍从号池租。

## 2. config.yaml

放仓库根目录。顶层只有 `apps`，每个应用一项。**key 用英文字母开头的英文、数字、下划线**（`camera`、`shop_web`、`APP01` 都行），这个 key 同时是脚本里 `case.config.<KEY>` 的名字（所以要是合法的 Python 变量名）和 `pages/<KEY>/` 的目录名。

每个应用**必须有 `platform`**。**不要写 `app_id` / `project_id`**：同步时 Nexus 按 key 对齐或自动创建应用，并把映射留在 Nexus 清单里。

应用在不同环境的身份不同（测试包名、测试域名）时用 `envs` 分环境；环境名与 Nexus 项目里的环境名一致（如 `test`、`prod`）：

```yaml
apps:
  camera:
    platform: android
    envs:
      test:
        package: com.example.a.test
        otp: {code: "123456"}    # 可选固定验证码；不写则取码时要问人
      prod: {package: com.example.a}
  shop_web:
    platform: web
    envs:
      test: {url: https://test.example.com}
      prod: {url: https://example.com}
  camera_ios:                # 各环境身份相同，可以不写 envs
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

### 固定验证码 `otp.code`

| 写法 | 含义 |
|---|---|
| 不写 `otp`，或不写 `otp.code` | 没有固定码；`case.scout.otp(...)` 会 `otp_requires_human`，agent 停下向人要本次验证码 |
| `otp: {code: "123456"}` | 直接用这个固定码（仅适合测试环境、可轮换的号） |

`otp` 可写在应用顶层，或写在某个 `envs.<环境>` 里（与包名 / URL 一起按环境合并）。**同步给 Nexus 时不要带 `otp`**（会被拒绝）；它只留在 Git，由 Scout 拉取后读取。

规则：

- 值从用户给的信息、用例前置条件或已有 `config.yaml` 里取；**拿不到包名 / URL / 环境就问用户，不要猜**；
- 已有应用的 key 不要改，改了会让 `pages/` 目录和已有用例对不上；
- 不放账号、密码、token（账号由 Nexus 号池租用）；测试用固定验证码仅可写在 `otp.code`；
- 新增应用 = `config.yaml` 加一项 + 建 `pages/<KEY>/` + push 后 `repo_apps_sync`（Nexus 自动建/对齐 app）；
- 应用的 `platform` 必须等于本次运行的设备平台；
- 本次运行的环境由 Nexus 决定，脚本里是 `case.env`，一般不需要判断；
- 这是应用地址与固定码的唯一真源，由 Scout 读取；Workbench 里的包名 / URL 只读展示，要改就改这个文件并 push，再按下面的约定同步给 Nexus。

## 3. 改了 config.yaml 必须同步给 Nexus

Nexus 不读脚本仓库（仓库可能是私有的），它只能靠你告诉它“这个项目的仓库能跑哪些应用”，Workbench 的“脚本仓库应用”和选应用都依赖这份同步。所以**只要新增、删除或修改了 `config.yaml` 里的应用或环境**（不含仅改 `otp.code` 也可同步身份字段；但 **sync 载荷里绝不能带 `otp`**），在 push 之后：

1. 取 push 后的提交：`git rev-parse HEAD`；
2. 调 MCP 工具 `repo_apps_sync(project_id, revision=<sha>, branch=<分支>, apps=<本次新增或修改的应用>, remove=<本次删除的应用 key>)`。这是差异更新：只传变化的应用（取自 `config.yaml` 的 **platform / package / bundle_id / url / envs**，**去掉 `otp`**，不要自己编 `app_id`），没传的应用保持不变；删除应用要显式放进 `remove`；
3. 调 `repo_apps_get(project_id)` 确认同步成功，`revision` 与刚 push 的一致，并记下 Nexus 回填的 `app_id`（跑数、选设备用）。

规则：

- 先 push 再同步，同步的 `revision` 必须是已 push 的提交；
- 未带 `app_id` 时 Nexus 按 key 对齐项目里已有同名应用，或自动创建；
- 同步被拒绝（key 命名、platform、字段不合法、误传 otp）就按报错改，改好重新 push 再同步；
- 只改了脚本、没动应用身份字段，不需要同步；只改了 `otp.code`、身份字段没变，也不需要同步；
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

每格 `selector` 里有 `kind` 和 `value` / `name`。`value` 必须是设备界面节点上的原文（`resource_id`、`text` 或 `content_desc`），不能用用例正文或 `label` 编一个能解析的选择器。还没有节点时，这一格先不写。`on_page` 用 `states[].anchors` 与界面文字比对，**一组锚点全部出现**才算在这一页；锚点也必须是这一屏真实出现过的文字。同一控件要在多个渠道用，就在 `locator_spec` 里写多格，不要在脚本里按平台分支。
