---
name: onboarding
description: 让 Mino 的任务能跑起来所需的前置步骤：在 Workbench 注册登录、用 OAuth 把 agent 接到 MCP、给有设备的电脑安装 Scout 节点并用 MCP 补齐 Chromium 等渠道程序、准备项目与脚本仓库。用户第一次使用 Mino、agent 连不上 Nexus（没有 Mino 工具或需要授权）、或要新增一台测试电脑时使用。
---

# 上手：注册、接入、装节点

写用例和跑用例之前，下面四件事必须先完成。每一步都在 **Workbench**（浏览器）里操作，你负责带用户走完并核对结果。

## 谁来操作

注册、在授权页点允许、复制安装命令这几步在 **浏览器**里完成，由**用户操作**。你的工作是给出完整网址和每一步要点什么，等用户确认，再核对结果。

- 邮箱和密码属于用户：不要替用户编邮箱和密码，不要把密码写进文件、记忆或回复；
- 页面和说明对不上、要二次验证、遇到验证码：让用户告诉你现在看到了什么，不要猜、不要反复重试；
- 节点装好之后，Chromium 等组件的安装**不需要再开网页**，用 MCP 工具即可（见「三、给有设备的电脑装 Scout」的最后一步）。

## 敏感信息

Scout 安装命令里的一次性安装凭证只在生成时显示一次，等同密码：

- 只用于安装那一台节点，不要贴进回复正文、日志、提交或记忆；
- 不要去读数据库、配置目录或浏览器存储来找它；过期了就在 Workbench 重新生成安装命令；
- 用完告诉用户“已接入”，不要复述命令里的凭证。

## 线上地址

| 用途 | 地址 |
|---|---|
| Workbench（网页） | `https://workbench.minocc.top` |
| Nexus（API / MCP） | `https://nexus.minocc.top` |
| MCP | `https://nexus.minocc.top/mcp` |

下面用 `<WORKBENCH>` = `https://workbench.minocc.top`，`<NEXUS>` = `https://nexus.minocc.top`。

## 一、注册并登录

1. 打开 `<WORKBENCH>/register`，填邮箱和密码（密码至少 8 位），提交；
2. 注册成功会跳到 `<WORKBENCH>/login`，用同一邮箱密码登录；
3. 已有账号直接去 `<WORKBENCH>/login`。

确认：登录后进入 Workbench 首页（项目列表）。

## 二、用 OAuth 接入你自己

MCP 地址：`https://nexus.minocc.top/mcp`。

Claude Code：

```bash
claude mcp add --transport http mino 'https://nexus.minocc.top/mcp'
```

其他客户端写成：

```json
{ "mcpServers": { "mino": { "type": "http", "url": "https://nexus.minocc.top/mcp" } } }
```

客户端连上后会打开浏览器。页面要求登录时，用刚才的 Workbench 账号登录，然后点允许。允许之后，这个客户端就以该账号调用 Mino。客户端自己保存并刷新凭证。

重启会话（或在 agent 里刷新 MCP 连接），确认出现名字以 `agent_guide`、`project_list`、`case_list` 开头的 Mino 工具。**第一件事调 `agent_guide`**，它是总入口。

确认：`project_list` 能返回项目（空列表也算通）。客户端停在需要授权：重新连接，让用户在浏览器里点允许。授权记录在 `<WORKBENCH>/assistant?tab=mcp`，可以吊销某一个客户端。助手功能没启用时告诉用户让管理员处理。

## 三、给有设备的电脑装 Scout

Scout 装在**有被测设备的那台电脑**上，不一定是你所在的电脑。

1. 打开 `<WORKBENCH>/settings/runtime`（节点与设备页），点“添加节点”；
2. 选这台电脑的系统（macOS / Linux 终端，或 Windows PowerShell）；
3. 页面生成安装命令（只显示一次，一段时间后过期，过期了点“生成新的安装命令”）；
4. 在**那台电脑**的终端里执行：
   - 这台电脑就是你所在的电脑：向用户说明命令会安装并注册系统服务，**得到用户同意后**你再执行；
   - 不是：把命令交给用户，让他在那台电脑上执行，你等待；
5. 回到页面看第 3 步“确认节点接入”，节点出现在节点列表且在线。

本机自检（在装 Scout 的电脑上）：

```bash
mino-scout status     # 服务是否在跑
mino-scout probe      # 能用的渠道和设备
```

### 装好节点后：用 MCP 补齐渠道程序

Scout 的下载中心已经通过 MCP 开放，不用回 Workbench 点：

1. `scout_list` 取节点：确认 `node_id`、在线状态、版本；
2. `plugin_status(node_id)` 看这台节点上各组件的 `installed` / `installable` / `install_state`；
3. 按要测的平台补齐：

| 要测的 | 工具 | 参数 |
|---|---|---|
| Web（浏览器） | `node_channel_program` | `node_id`、`tool_id=chromium`、`action=install` |
| Android 输入 | `node_channel_program` | `tool_id=adb.keyboard` |
| iOS | `node_channel_program` | `tool_id=appium` |
| 飞书 / Meego 工作项、对话渠道 | `plugin_install` | `plugin=cli.feishu` / `cli.meego` / `langbot` |

规则：节点必须在线；`operation_id` 重试时沿用同一个，避免重复安装；返回任务还没完成就用 `plugin_status(node_id)` 按任务查状态，不要换请求 ID 重装；Nexus 不下载也不保存安装包，是节点自己下载。

其余依赖（`adb` 本身、WDA、桌面授权等）见 `overview` 的“安装 Scout”一节。装完设备仍看不到：让用户去 `<WORKBENCH>/settings/runtime` 看节点和设备状态，不要自己改配置文件。

## 四、项目、应用和脚本仓库

1. 项目：在 `<WORKBENCH>/testing` 创建或选择一个项目（`project_list` 能看到）；
2. 脚本仓库：在该项目的配置页，**“脚本仓库”卡片**里填仓库地址和分支（项目的 Git 仓库；私有仓库要在装 Scout 的电脑和你的电脑上各自配好 git 凭据，Nexus 不保存令牌）；
3. 环境：在 Workbench 项目里确认有哪些环境（如 `test`、`prod`）。加应用前**先问用户**环境清单；后面 `config.yaml` 的环境名必须与之一致；
4. 应用：用户要加安卓 / iOS / Web 等应用时，**整段按 `repo-config` §0**：先问平台、各环境的包名 / bundle id / URL，再写 `config.yaml`（不要写 `app_id`），push 后 `repo_apps_sync`。Nexus 按 key 对齐或自动创建。**不要**只调 `app_create` 就说做完——那个只建空壳，没有身份字段不能跑；
5. 用例：在 Workbench 导入或创建用例（`case_list` 能取到）。

确认：`project_list` 返回的项目带 `repo_url`、`repo_branch`；`repo_apps_get` 能看到已同步的应用，且带上包名 / URL 等身份字段（不只是名字）。

MCP 只通信，不写脚本；脚本由 agent 在本机工作区编写并通过 Git 推送。

## 做完之后

按顺序使用：`repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。Mino 各部分怎么分工见 `overview`。技能本身需要更新时见 `update-skills`。
