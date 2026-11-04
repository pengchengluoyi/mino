---
name: onboarding
description: 让 Mino 的任务能跑起来所需的前置步骤：在 Workbench 注册登录、生成 MCP token 接入 agent、给有设备的电脑安装 Scout 节点并用 MCP 补齐 Chromium 等渠道程序、准备项目与脚本仓库。用户第一次使用 Mino、agent 连不上 Nexus（没有 Mino 工具或 401）、或要新增一台测试电脑时使用。
---

# 上手：注册、接入、装节点

写用例和跑用例之前，下面四件事必须先完成。每一步都在 **Workbench**（浏览器）里操作，你负责带用户走完并核对结果。

## 谁来操作

注册、生成 token、复制安装命令这几步在 **Workbench 网页**里完成，由**用户在自己的浏览器里操作**。你的工作是给出完整网址和每一步要点什么，等用户确认，再核对结果。

- 邮箱和密码属于用户：不要替用户编邮箱和密码，不要把密码写进文件、记忆或回复；
- 页面和说明对不上、要二次验证、遇到验证码：让用户告诉你现在看到了什么，不要猜、不要反复重试；
- 节点装好之后，Chromium 等组件的安装**不需要再开网页**，用 MCP 工具即可（见「三、给有设备的电脑装 Scout」的最后一步）。

## 敏感信息

MCP token 和 Scout 安装命令（里面带一次性安装凭证）都**只在生成时显示一次**，等同密码：

- 只用于下面的接入操作，不要贴进回复正文、日志、提交或记忆；
- 不要去读数据库、配置目录或浏览器存储来找 token；丢了就在 Workbench 重新生成；
- 用完告诉用户“已接入”，不要复述 token 内容。

## Nexus 地址

向用户确认 Nexus 的地址：

| 环境 | 地址 |
|---|---|
| 正式部署 | `http://<服务器>:50104`（Workbench 内置在 Nexus 里，同一个地址） |
| 本机源码开发 | `http://127.0.0.1:50101`（前端开发服务器 `http://localhost:60101`） |

下面用 `<NEXUS>` 表示它。

## 一、注册并登录

1. 打开 `<NEXUS>/register`，填邮箱和密码（密码至少 8 位），提交；
2. 注册成功会跳到 `<NEXUS>/login`，用同一邮箱密码登录；
3. 已有账号直接去 `<NEXUS>/login`。

确认：登录后进入 Workbench 首页（项目列表）。

## 二、生成 MCP token，接入你自己

1. 打开 `<NEXUS>/assistant?tab=mcp`（助手页的 MCP 区块）；
2. 点生成 token，**起一个能认出用途的名字**（如 `claude-code-<电脑名>`），确认；
3. 页面显示一次接入命令，形如 `claude mcp add --transport http mino '<MCP 地址>' --header 'Authorization: Bearer <token>'`（也有 JSON 配置形式，给其他客户端用）；
4. 你是 Claude Code 就在终端执行这条命令；其他 agent 工具用 JSON 配置写入它自己的 MCP 配置；
5. 重启会话（或在 agent 里刷新 MCP 连接），确认出现名字以 `agent_guide`、`project_list`、`case_list` 开头的 Mino 工具。**第一件事调 `agent_guide`**，它是总入口。

确认：`project_list` 能返回项目（空列表也算通）。返回 401：token 无效或被吊销，回到第 1 步重新生成；token 页打不开或提示未就绪：助手功能没启用，告诉用户让管理员处理。

## 三、给有设备的电脑装 Scout

Scout 装在**有被测设备的那台电脑**上，不一定是你所在的电脑。

1. 打开 `<NEXUS>/settings/runtime`（节点与设备页），点“添加节点”；
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

其余依赖（`adb` 本身、WDA、桌面授权等）见 `overview` 的“安装 Scout”一节。装完设备仍看不到：让用户去 `<NEXUS>/settings/runtime` 看节点和设备状态，不要自己改配置文件。

## 四、项目、应用和脚本仓库

1. 项目：在 `<NEXUS>/testing` 创建或选择一个项目（`project_list` 能看到）；
2. 脚本仓库：在该项目的配置页，**“脚本仓库”卡片**里填仓库地址和分支（项目的 Git 仓库；私有仓库要在装 Scout 的电脑和你的电脑上各自配好 git 凭据，Nexus 不保存令牌）；
3. 应用：用 `app_create` 或在 Workbench 创建，得到应用 id，再按 `repo-config` 写 `config.yaml` 并同步；
4. 用例：在 Workbench 导入或创建用例（`case_list` 能取到）。

确认：`project_list` 返回的项目带 `repo_url`、`repo_branch`。

## 做完之后

按顺序使用：`repo-config` → `write-test-cases` → `check-test-cases` → `run-test-cases`。Mino 各部分怎么分工见 `overview`。技能本身需要更新时见 `update-skills`。
