---
name: onboarding
description: 第一次让 Mino 能跑：注册登录、OAuth 接 MCP、用 MCP 申请安装命令并经确认后在本机安装 Scout、再按固定顺序建项目 / 绑脚本仓 / 配应用。用户首次使用、缺 MCP、本机无 mino-scout、或要新增节点 / 新项目时使用。
---

# 上手（按门禁推进，不要跳步）

目标：本机 Scout 在线 + MCP 可用 + 项目已绑脚本仓 + 应用清单已同步。  
**每完成一节，用一两句话告诉用户本节结果和下节要做什么，再进入下一节。**

工具分工见 `overview`：**本机用 `mino-scout` CLI；云端用 MCP；仓用 Git。**

## 谁操作什么

| 动作 | 谁 |
|---|---|
| 注册 / 登录 / OAuth 点允许 | **用户**（浏览器） |
| 申请 Scout 安装命令、建项目、绑仓、同步应用、装渠道程序 | **你**（MCP） |
| 在本机执行安装（须用户确认弹窗同意）、`probe` / `check`、写仓 push | **你**（本机） |
| 安装命令里的一次性凭证 | 只用于当次安装；**不要**写入回复正文、日志、记忆、提交 |

页面和说明对不上、要验证码：让用户描述当前画面，不要猜、不要狂重试。

## 线上地址

| 用途 | 地址 |
|---|---|
| Workbench | `https://workbench.minocc.top` |
| Nexus / MCP | `https://nexus.minocc.top` / `https://nexus.minocc.top/mcp` |

## 门禁 0：告诉用户整条路径

开场先说清将按下面顺序做，**用户未确认安装就不要执行安装命令**：

1. 注册登录（若尚未）  
2. Agent 接 MCP  
3. MCP 申请安装命令 → 确认弹窗 → 本机安装并验收 Scout  
4. 项目 → 脚本仓绑定 → 环境与应用 → push → 同步  

未完成门禁 1–3，不要写 `config.yaml`、不要 `case_run`、不要自建 GitHub 仓。

---

## 门禁 1：注册并登录

1. 用户打开 `https://workbench.minocc.top/register`（已有账号则 `/login`）；  
2. 密码 ≥8 位；你不代填、不保存密码。  

确认：用户进入 Workbench 首页。通过后告诉用户：「账号就绪，下一步接 MCP。」

---

## 门禁 2：OAuth 接入 MCP

MCP：`https://nexus.minocc.top/mcp`。

Claude Code 示例：

```bash
claude mcp add --transport http mino 'https://nexus.minocc.top/mcp'
```

其他客户端：

```json
{ "mcpServers": { "mino": { "type": "http", "url": "https://nexus.minocc.top/mcp" } } }
```

用户在浏览器登录并点允许。重启会话或刷新 MCP 后：

1. 确认出现 Mino 工具；  
2. **先调 `agent_guide`**；  
3. 再调 `project_list`（空列表也算通）。  

授权页卡住：让用户点允许；记录在 `https://workbench.minocc.top/assistant?tab=mcp`。

通过后告诉用户：「MCP 已通，下一步装 / 验 Scout。」

---

## 门禁 3：安装并验收 Scout

Scout 装在**有被测设备的那台电脑**上。

### 3.1 先查本机是否已有

```bash
command -v mino-scout
mino-scout
mino-scout probe
```

已有且 `probe` 能看到目标平台设备 → 本节完成，跳到门禁 4。  
否则进入 3.2。

### 3.2 同机安装（MCP 申请命令 → 确认 → 执行）

不要让用户去 Workbench 抄命令，也不要让用户把命令贴回给你。

1. 识别本机 OS（`darwin` / `linux` / `win32`）。  
2. 调 MCP **`scout_install_token`**（对应 `POST /runtime/nodes/install-token`）：拿到 `install_id`、`expires_at`、各系统 `commands`。  
3. 向用户说明：将在本机安装 Scout 并注册系统服务、连到当前账号；**不要**在对话里复述 token 或完整命令原文。  
4. 取出与本机 OS 匹配的那条命令，提交本机执行；由客户端**确认弹窗**让用户同意后再跑。用户拒绝则停止，不要改走「请粘贴命令」。  
5. 用 `install_id` 查安装状态（若 MCP 提供查询；否则 `scout_list` + 本机 CLI）直到节点出现。  
6. 本机验收：

```bash
mino-scout
mino-scout probe
```

7. MCP：`scout_list` 节点 `online`；`device_list` 对照设备。  
8. 渠道程序（节点在线后，MCP）：

| 要测的 | 调用 |
|---|---|
| Web | `node_channel_program(node_id, tool_id=chromium, action=install)` |
| Android 键盘输入 | `node_channel_program(..., tool_id=adb.keyboard)` |
| iOS | `node_channel_program(..., tool_id=appium)` |
| 飞书 / Meego CLI | `plugin_install(node_id, plugin=cli.feishu 或 cli.meego)` |

先 `plugin_status(node_id)`；缺什么装什么。`operation_id` 重试沿用同一个。  
其余本机依赖（`adb`、WDA、macOS 录屏 / 辅助功能）缺了告诉用户去系统设置授权，**不要改 Scout 配置文件**。

### 3.3 设备在另一台电脑

在**那台电脑**上由那边的 agent 走 3.2；你这边只确认 `scout_list` / `device_list`。不要把安装凭证贴进聊天让用户手抄。

通过后告诉用户：「Scout 已就绪，下一步配置项目与脚本仓。」

---

## 门禁 4：项目 → 脚本仓 → 应用（固定顺序）

**禁止跳步。禁止用户未要时新建远程 Git 仓。**  
每一步开始前说清要做什么、需要用户确认什么。

### 4.1 项目

1. `project_list` 看是否已有目标项目。  
2. 没有：复述名称（和可选说明）→ 用户确认后调 **`project_create(name, description?)`** → 记下 `project_id`。  
3. 告诉用户项目已选定。

### 4.2 脚本仓绑定

一次问清：脚本仓 **Git URL**、分支名。  
用户只说「建项目 / 配应用」→ 只问仓地址，不要 `gh repo create`。  
仅当用户明确要求新建远程仓时，先复述名字与可见性，得同意后再建。  
私有仓：本机与 Scout 所在机都要有 git 凭据；Nexus 不存令牌。

```text
project_script_repo_set(project_id, url, branch)
```

确认：`project_list` 出现 `repo_url` / `repo_branch`。  
本地：`git clone`（或打开已有检出）**这个 URL**，后续只在该工作副本改文件、commit、push。

### 4.3 环境与应用身份

问清并复述确认：

| 项 | 说明 |
|---|---|
| 环境名 | 与项目一致，常见 `test` / `pre` / `prod` |
| 每个应用一条 | 一平台一 key（如 `baidu_android`、`baidu_web`） |
| 身份字段 | Android `package`；iOS `bundle_id`；Web `url` |

然后按 `repo-config`：

1. 在已绑定仓根写 / 改 `config.yaml`（不写 `app_id` / `project_id`）；  
2. 建 `pages/<KEY>/`；  
3. `mino-scout check .`；  
4. commit + push；  
5. `repo_apps_sync(...)`（不要传 `otp`）；  
6. `repo_apps_get` 确认。  

**不要**只调 `app_create` 就说完成。

### 4.4 需求与用例（写脚本前先弄清关系）

| | 需求 requirement | 用例 case |
|---|---|---|
| 是什么 | 一块要测的功能 / 文档范围 | 该范围内**一条**可执行检查 |
| Nexus | `requirement_save` 登记：`requirement_id`、标题、`source_url` | 一般不存步骤正文；靠脚本仓文件 |
| 磁盘 | 目录 `tests/<requirement_id>/` | 文件 `tests/<requirement_id>/<case_id>.py` |
| 列表工具 | `case_list` / `requirement_list` 返回已登记需求的 id、名称、链接 | 用来知道「测哪块、原文在哪」 |

流程：`requirement_save` → 打开 `source_url`（或用户给的原文）读出要测的步骤 → 写成多条用例脚本。  
**列表接口不提供用例步骤正文**；正文来自链接 / 用户材料，落盘在 `tests/.../*.py`。

---

## 本节完成态

- [ ] MCP：`project_list` 可用  
- [ ] CLI：`mino-scout` / `probe` 正常，目标设备可见  
- [ ] 项目存在且 `repo_url` / `repo_branch` 已设  
- [ ] `repo_apps_get` 能看到应用身份字段  
- [ ] 本地工作副本的 `git remote` 就是该项目绑定的脚本仓  

之后：`write-test-cases` → `check-test-cases` → `run-test-cases`。  
技能更新见 `update-skills`。
