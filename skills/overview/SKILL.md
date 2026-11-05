---
name: overview
description: 了解 Mino 是什么、Scout / Nexus / Workbench 三个仓库各自负责什么、Scout 怎么安装到本地电脑。用户问“这个系统怎么分工”“Scout 是什么”“怎么装 Scout”，或你不确定该找哪一端时使用。
---

# Mino 总览

Mino 是一套自动化测试系统：由 AI 写测试脚本，在真实设备（手机、浏览器、桌面应用）上执行，并留下证据。三个仓库各管一件事，**互相不越界**。

| 仓库 | 是什么 | 负责 | 不负责 |
|---|---|---|---|
| **MinoNexus** | 服务端，大脑 | 用例、项目与仓库配置、设备与账号资源、任务与会话、事件存储、触发任务并把仓库地址 / 提交 / 设备下发给 Scout、对 Workbench 和 agent（MCP）供数 | 直接操作设备；读写或打包脚本仓库；保存 Git 令牌 |
| **MinoScout** | 执行器，手脚，装在有设备的电脑上 | 接收 Nexus 下发的任务，**自己用本机 git 拉取脚本仓库**、读 `config.yaml`、跑脚本，操作本机设备，截图取证，回报结果；本机取验证码和调视觉模型 | 存业务数据；选用例；分配账号；决定“下一步做什么” |
| **MinoWorkbench** | 唯一的前端（网页），内置在 Nexus 里 | 管理项目、用例、环境、控件，查看任务、会话和证据 | 执行、写脚本 |

Scout 永远是主动连向 Nexus 的一方，不开放监听端口。一台 Scout 可以挂多台本机设备。

## 一次测试的流向

1. 用例在 Nexus（Workbench 里维护）；
2. agent 通过 Nexus 的 MCP 工具读用例，在本机写脚本并 push 到项目的 Git 仓库；
3. agent 调 `case_run`，Nexus 只触发任务：把仓库地址、分支、提交、环境、设备下发给设备所在的 Scout；
4. Scout 自己拉取该提交的脚本仓库（用本机 git 凭据），在设备上执行，步骤和证据写回 Nexus；
5. 用户在 Workbench 看过程和结果，agent 在对话里简要汇报。

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

## 安装 Scout 到本地电脑

Scout 装在**有被测设备的那台电脑**上（Android 手机接的电脑、跑浏览器的电脑、要测的 Mac / Windows 机器）。

1. 安装命令由 **Workbench 生成**：在 `/settings/runtime`（节点与设备）点“添加节点”，命令里带 Nexus 地址和一次性安装凭证，具体操作见 `onboarding`。命令会运行随 GitHub Release 发布的 `bootstrap.sh`（macOS / Linux）或 `bootstrap.ps1`（Windows PowerShell），自动下载对应系统和架构的安装包并注册服务；
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

细节以 MinoScout 仓库的 `docs/DEVICE_SETUP.md` 为准。拿不到安装命令、节点不在线、`probe` 里看不到设备时，**告诉用户去 Workbench 查看节点状态，不要自己改配置文件或编造命令**。

## 安装 Nexus / Workbench

Workbench 已内置在 Nexus 里，一个服务一个端口（默认 50104），浏览器直接访问。Nexus 的安装和升级见 MinoNexus 仓库 `docs/github-updates.md`，这通常由管理员完成，不属于写用例的范围。
