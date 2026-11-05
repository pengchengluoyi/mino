---
name: write-test-cases
description: 为 MinoScout 编写用例脚本（Python）。用户要求为某条用例写脚本、改脚本时使用。仓库目录和 config.yaml 见 repo-config，写完检查见 check-test-cases，执行见 run-test-cases。
---

# 编写用例脚本（Scout）

脚本是 Python，一条用例一个文件，开跑后整份在用户电脑上的 Scout 执行。仓库怎么组织、`config.yaml` 怎么配、页面文件怎么写，见 `repo-config`，本技能只讲**用例里怎么写**。

方法签名以本机安装的运行时为准，不确定时读源码，不要凭记忆：

```bash
python -c "import mino_scout, pathlib; print(pathlib.Path(mino_scout.__file__).parent / 'runtime' / 'script_exec.py')"
```

## 1. 写法

设备动作不挂在 `case` 上。先把被测应用实例化，再通过实例操作：

```python
def run(case):
    camera = case.init(case.config.camera)         # 绑定 config.yaml 里的 camera
    with case.nexus.prep("清理应用缓存"):            # 前置，写在所有 do 之前
        camera.clear_cache()
    with case.nexus.do("2-1 启动应用"):              # 原文第 2 步拆开，序号保留
        camera.launch()
    with case.nexus.do("2-2 点击快拍按钮"):
        camera.tap("splash.quick_snap")             # page_key.control_id，页面在 pages/camera/
    with case.nexus.check("出现取景框"):
        camera.on_page("viewfinder")
        camera.vlm("画面里有取景框", verdict="pass")
```

三类对象：

| 对象 | 需要实例化 | 方法 |
|---|---|---|
| `app`（`case.init(...)` 的返回值） | 是，一个应用一个 | 见 §3 |
| `case.scout` | 否 | `wait(ms)`、`otp_mail()`、`llm(prompt, text, schema)` |
| `case.nexus` | 否 | `prep(title)` / `do(title)` / `check(title)`、`account()`、`note(text, **fields)`、`fail(message)` |

`case` 上只有 `config`、`env`、`init`、`scout`、`nexus`。`case.tap(...)`、`case.launch()` 这类写法不存在。写错方法名，运行时会报出相近的名字和可用列表，按提示改。

## 2. 步骤事件就是测试报告

`with case.nexus.prep/do/check(title):` 把“进入步骤、离开步骤、成败”写入 Nexus 会话，Workbench 按它展示测试过程和证据。所以：

- `title` 写**这一步具体做什么 / 验证什么**的一句话，取用例原文的意思，例如 `do("输入手机号并点击获取验证码")`、`check("提示验证码已发送")`；
- 不要写 `do("步骤1")`、`do("操作")`，报告里会看不出哪一步坏了；
- 前置、操作、预期都要落进脚本，并且**至少**覆盖原文的每一条：每条前置至少一个 `prep`，每条操作至少一个 `do`，每条预期至少一个 `check`。一条原文可以拆成相邻的几步，标题用原文序号加子号，例如第 2 条操作写成 `do("2-1 打开登录页")`、`do("2-2 输入手机号")`，预期和前置同样用 `1-1`、`2-1`。顺序跟原文一致，先写完前置再写操作；
- 拆步是把一条原文分成几次动作，不是另编原文里没有的步骤。原文缺步骤、对不上或不可判定就停下，并写下差异，不猜；
- 前置写在同文件的 `run`（或同文件的 `pretest`）里，放在所有 `do` 之前。`precondition` 为空、并且确实不用清缓存、登录、安装或授权时，仍然写一条 `prep`，块里可以没有动作，标题写清为什么不做，例如 `prep("不依赖缓存或登录态，安装后直接开始")`。这条标题会写入 Nexus。不要写 `prep("无")`；
- 前置里的「清理缓存 / 清除缓存 / 清除应用数据」，Android 用 `app.clear_cache()`。iOS 上这个方法会 `declined`，不要换别的命令绕过去，做不到就停下问；
- 一个 `with` 块里只放这一步的动作；失败时 Workbench 定位到这个 title；
- 补充证据用 `case.nexus.note("文字", 键=值)`，不要 `print`；
- 账号一律 `case.nexus.account()` 取（Nexus 号池租用），仓库任何文件都不写账号、密码、验证码。

## 3. `app` 的方法与渠道支持

方法名在所有渠道相同，但**不是每个渠道都支持**。不支持的调用会失败并带 `reason_code=declined`，不要用别的写法绕：

| 方法 | android | ios | web | mac | win |
|---|---|---|---|---|---|
| `launch()` `close()` `kill()` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `tap(c)` `input(c, text)` `long_press(c)` `swipe(dir)` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `tap_xy(x, y)` `drag(x1,y1,x2,y2)` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `press(key)` `back()` | ✓ | 仅 home / volume_up / volume_down | ✓ | ✓ | ✓ |
| `set_clipboard(text)` | ✓（部分机型不支持） | ✓ | ✓ | ✓ | ✓ |
| `foreground()` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `exec(script, timeout_ms)` | 设备 shell | 本机命令 | 页面 JS | zsh | PowerShell |
| `install(path)` | ✓ | ✓ | – | ✓ | ✓ |
| `uninstall()` `version()` | ✓ | ✓ | – | – | – |
| `clear_cache()` | ✓ | declined | – | – | – |
| `read_data(key)` `set_ime(ime)` | ✓ | – | – | – | – |
| `reload()` `open_tab(url)` `switch_tab(i)` `upload(path, c)` `wait_ready(ms, c)` | – | – | ✓ | – | – |
| `screenshot()` `sample(ms, interval)` `hierarchy()` `on_page(page)` `vlm(...)` `vlm_frame(i, ...)` | ✓ | ✓ | ✓ | ✓ | ✓ |

`c` 表示控件，写 `page_key.control_id`。所有方法返回 `ActionResult`（`ok`、`summary`、`status`、`data`），`vlm` 系列返回字符串。

## 4. 视觉判定与坐标

- `app.vlm(prompt)` 看**这个应用**最近一张截图（没有就自动截一张），返回模型描述的文字；
- `app.vlm(prompt, verdict="pass")`：模型判定未通过会直接中止，`reason_code=vlm_failed`；
- `app.sample(duration_ms, interval_ms)` 连续截图，再用 `app.vlm_frame(i, prompt)` 看第 i 帧；
- 截图只在 Scout 本机和模型之间传递，不回 Nexus；
- `tap_xy` / `drag` 只在控件定位不了、必须依据视觉结果点位置时才用；能用 `pages` 控件就不要用坐标。`vlm` 不返回节点，也不能当成 `locator_spec`。

## 5. 规则

- `case.init(...)` 的参数只能写 `case.config.<KEY>`，不写字面量包名 / 地址；
- 多个应用就多个实例，各管各的页面和截图；同一用例里顺序执行，不并行；
- 控件写 `page_key.control_id`，页面里没有的控件不点；
- `locator_spec` 的值必须是这台设备当前界面节点上的 `resource_id`、`text` 或 `content_desc`，逐字复制。用例正文、按钮的中文说法、页面 `label` 都不是选择器。没有这份节点时，页面只留 `id` 和 `label`，不要写 `locator_spec`，也不要写对应的 `tap` / `input`；
- 同一屏上这个值出现多次就不能用。优先唯一的 `resource_id`，其次唯一的 `content_desc`，最后才是唯一的 `text`。不可点击的标题和可点击的按钮文案相同，只算可点击的那一个。页面锚点同样只取这一屏真实出现过的文字；
- 用例叫法能对上某个节点的 `text` 或 `content_desc`，定位就用这份文字。对不上时，用这一步截图判断目标在画面的哪一块，只在该区域的真实节点里比较 `locator_spec`。视觉给出的位置不写入 `pages`，也不写 `bounds` 或 `tap_xy`。哪一份定位匹配度最高，用下一次 `case_run` 点一下再读界面来定，不能把视觉结果原样落成选择器；
- **不要自己再造 Scout 已有的能力**：

| 不要写 | 改用 |
|---|---|
| `time.sleep(...)` | `case.scout.wait(ms)` |
| `imaplib` / `poplib` 取验证码 | `case.scout.otp_mail()` |
| `openai` / `anthropic` 调模型 | `case.scout.llm()` / `app.vlm()` |
| `subprocess`、`os.system`、`exec`、`eval` | `app.exec()` |
| `adbutils`、`playwright`、`appium` 等设备库 | `app.*` |
| `threading`、`multiprocessing`、`asyncio` | 顺序写，不并发（线程挂住会拖垮整个节点） |
| `if platform == ...` | 差异放进 `pages` 的 `locator_spec` |

执行时**不会**拦截这些写法，但它们会让脚本绕开取证、超时和取消，所以写完一定用 `check-test-cases` 自检并按提示改掉。

## 6. 编写流程

页面参数、点击、截图、界面结构都只来自 Scout 执行这条用例脚本。`app.hierarchy()`、`app.screenshot()`、`app.tap()`、`app.input()` 就是 Scout 的能力。写的时候调用 `case_run`，由 Scout 跑 `script.run`。不要用浏览器、adb、Playwright 或其他工具去看界面、点控件、抄 DOM。

1. 取用例：`case_list(project_id, search=<case_id>)`，拿到 `case_id`、`requirement_id`、`platform`、`steps`、`expected`、`precondition`。**步骤不连续、平台不明确、预期不可判定就停下问，不猜。**
2. 取仓库：`project_list` 拿 `repo_url`、`repo_branch`。
   - 本机路径约定 `~/.mino/workspaces/<project_id>`；
   - 不存在：`git clone --branch <repo_branch> <repo_url>`；
   - 已存在：`git fetch` 后 `git pull --ff-only`；
   - 有未提交改动、有未 push 提交、或 ff 失败：**停下问用户**，不 `reset --hard`，不 force。
3. 选设备：按 `run-test-cases` 调 `device_list`，只选 `executable=true` 且平台与用例一致的一台。没有就停下，**不要按用例正文编 `locator_spec`**。写这一条期间一直用这台设备。
4. 按 `repo-config` 补目录。缺应用就先补 `config.yaml` 和 `pages/<KEY>/`。**改了 `config.yaml` 的话，push 后要调 `repo_apps_sync`**（见 `repo-config` §3）。
5. 在 `tests/<需求ID>/<case_id>.py` 里用 Scout 把路径走通，再落成可执行脚本。同一份文件改一轮、跑一轮：
   - 还没有真实定位时，脚本只做到达这一屏所必需的事，并调用 `app.hierarchy()` 和 `app.screenshot()`。能用已写好的 `page_key.control_id` 就 `tap` / `input`，没有定位就不点；
   - `check-test-cases` 通过后 commit 并 push，用上一步的设备 `case_run`（`run-test-cases`）；
   - 从这次会话的 `tool/result` 取界面：`capability_id=hierarchy` 的 `nodes`。没有 `nodes` 时才读同条事件的 `hierarchy_text`，并且只抄里面真实出现的属性。截图用 `session_screenshot`；
   - 一步做完不要只读一帧。用 `app.sample` 连续截几秒，并在这段时间里多次 `app.hierarchy()`。页面可能过一会儿才换。只凭一次读取就说没进到目标页，证据不够；
   - 用这些节点写 `pages/<APP_KEY>/<page_key>.yaml`，只写测过的 `locator_spec`。文字对得上就用文字。对不上时，截图只用来圈定区域，在该区域节点里选出匹配度最高的定位，再用下一次 `case_run` 核对。截图上的坐标不进页面文件；
   - 把对应用例原文的 `prep` / `do` / `check` 补进脚本，只点页面里已经有定位的控件；
   - 一步不是单个控件能完成时（登录填表、滑到底再确认、一组连续输入），写成 `flows/<name>.py`，函数接收 `app`，里面仍然只调用 `app.*`。纯计算、不碰设备的逻辑写成 `scripts/<name>.py`。用例里 `from flows...` 或 `from scripts...` 调用。这两处也要随同一轮 `case_run` 调试，不能只留页面文件；
   - 下一屏重复。原文走完后，脚本必须覆盖每一条前置、操作和预期，不能停在只采集界面的版本。`flows` / `scripts` 搞不定的那一步停下来问，不要换一套工具去点。
6. 写完再按 `run-test-cases` 用同一台设备 `case_run` 调试一遍。`locator_missing`、`control_unknown`、点错控件，回到上一步用这次运行的 hierarchy 改页面，不要按正文重写定位。通过后再向用户汇报。
7. 用例原文和当前界面走的不是同一条路时，差异写在对话里，不写进用例、不写进 `pages`、不用 `case.nexus.note` 当结论。写清：原文这一步要去哪、当前实际停在哪一屏、中间多出来或少掉什么。它可能是缺陷，也可能是后加的需求，所以不改原文。然后继续沿实际界面往下看，能对上原文后续步骤的就继续写；没走到的控件不要留成空的 `locator_spec`，等进入它该出现的那一页再写。
