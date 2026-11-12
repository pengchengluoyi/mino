---
name: overview
description: 了解 Mino 是什么、Scout / Nexus / Workbench 三端各自负责什么、Scout 怎么安装到有设备的电脑上。用户问“这个系统怎么分工”“Scout 是什么”“怎么装 Scout”，或你不确定该找哪一端时使用。
---

# Mino 总览

Mino 是一套自动化测试系统：由 AI 写测试脚本，在真实设备（手机、浏览器、桌面应用）上执行，并留下证据。三端各管一件事，**互相不越界**。

| 端 | 是什么 | 负责 | 不负责 |
|---|---|---|---|
| **Nexus** | 云端服务（`https://nexus.minocc.top`），大脑 | 用例、项目与仓库配置、设备与账号资源、任务与会话、事件存储、触发任务并把仓库地址 / 提交 / 设备下发给 Scout、对 Workbench 和 agent（MCP）供数；按脚本仓 key 对齐 `app_id` | 直接操作设备；读写或打包脚本仓库；保存 Git 令牌；托管固定验证码 |
| **Scout** | 执行器，手脚，装在有设备的电脑上 | 接收 Nexus 下发的任务，**自己用本机 git 拉取脚本仓库**、读 `config.yaml`（含可选 `otp.code`）、跑脚本，操作本机设备，截图取证，回报结果；本机取验证码和调视觉模型 | 存业务数据；选用例；分配账号；决定“下一步做什么” |
| **Workbench** | 网页前端（`https://workbench.minocc.top`），由 Nexus 提供 | 管理项目、用例、环境、控件，查看任务、会话和证据 | 执行、写脚本 |
| **Agent + MCP** | 对话侧 | agent 写脚本并 Git 推送；MCP 只查询、同步应用清单、触发跑数 | MCP 不写代码、不改脚本仓 |

用户电脑上安装的只有 Scout；本地状态（如脚本工作区）放在 `~/.mino/`。Scout 永远是主动连向 Nexus 的一方，不开放监听端口。一台 Scout 可以挂多台本机设备。MCP 入口：`https://nexus.minocc.top/mcp`。

## 一次测试的流向

1. 用例在 Nexus（Workbench 里维护）；
2. agent 通过 MCP 读用例，在本机写 `config.yaml` / 脚本并 push（脚本仓用 key，不写 `app_id`）；
3. 改了应用清单则 `repo_apps_sync`，Nexus 按 key 对齐或自动创建应用；
4. agent 调 `case_run`，Nexus 只触发任务：把仓库地址、分支、提交、环境、设备、`app_key` 下发给设备所在的 Scout；
5. Scout 自己拉取该提交的脚本仓库（用本机 git 凭据），在设备上执行，步骤和证据写回 Nexus；
6. 用户在 Workbench 看过程和结果，agent 在对话里简要汇报。

## 该用哪个技能

| 你要做的事 | 技能 |
|---|---|
| 搞清楚怎么分工、怎么装 Scout | 本技能 |
| 第一次使用：注册、用 OAuth 接入 MCP、给电脑装 Scout 节点、准备项目和仓库 | `onboarding` |
| 了解脚本仓库怎么组织、`config.yaml` 怎么配应用和环境 | `repo-config` |
| 为用例写脚本 | `write-test-cases` |
| 写完检查脚本 | `check-test-cases` |
| 执行用例、看结果、汇报 | `run-test-cases` |
| 更新技能、反馈技能的问题 | `update-skills` |

## 安装 Scout

Scout 装在**有被测设备的那台电脑**上（Android 手机接的电脑、跑浏览器的电脑、要测的 Mac / Windows 机器）。

1. 安装命令由 **Workbench** 生成：在 `/settings/runtime`（节点与设备）点“添加节点”，命令里带 Nexus 地址和一次性安装凭证，具体操作见 `onboarding`。命令会运行随 GitHub Release 发布的 `bootstrap.sh`（macOS / Linux）或 `bootstrap.ps1`（Windows PowerShell），自动下载对应系统和架构的安装包并注册服务；
2. 装完在 Workbench 的节点列表里确认这台 Scout 在线（安装成功只代表文件和服务就绪，是否接入以 Workbench 为准）；
3. 本机自检：

```bash
mino-scout status     # 服务状态
mino-scout probe      # 打印本机能用的渠道和设备，不连 Nexus
```

按要测的平台准备本机依赖：

| 平台 | 本机需要 |
|---|---|
| Android | `adb`，手机开 USB 调试并授权 |
| iOS | WDA（WebDriverAgent）可用，设备已连接 |
| Web | 已安装 Chromium（安装包自带 `ms-playwright`） |
| Mac / Windows 桌面 | 已登录的图形桌面会话；macOS 需给 Scout 授权屏幕录制和辅助功能 |

拿不到安装命令、节点不在线、`probe` 里看不到设备时，**告诉用户去 Workbench 查看节点状态，不要自己改配置文件或编造命令**。
