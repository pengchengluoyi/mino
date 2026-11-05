---
name: check-test-cases
description: 写完或改完用例脚本后，用 mino-scout check 自检并按提示修改。每写完一条用例都要运行一次。
---

# 自检用例脚本

执行时 Scout **不做任何预检**，脚本写成什么样都会跑。所以问题要在写的时候发现：每写完一条用例（或改完 `config.yaml`、`pages/`、`flows/`），在脚本仓库根目录运行：

```bash
mino-scout check .
```

输出每行一条提示：`文件:行号: [类型] 说明和改法`。有提示退出码是 1，没有是 0。**按提示改完，再运行一次，直到“没有提示”，再 commit 和 push。**

| 类型 | 意思 | 怎么改 |
|---|---|---|
| `forbidden` | 绕开了 Scout（设备库、`subprocess`、线程、`exec`/`eval`、`if platform`） | 设备动作改用 `app.*`，设备命令改用 `app.exec()`，不要并发，平台差异放 `pages` 的 `locator_spec` |
| `reuse` | 自己重复实现了 Scout 已有的能力（`time.sleep`、`imaplib`、直接调模型） | 改用 `case.scout.wait()` / `otp_mail()` / `llm()` / `app.vlm()` |
| `case_attr` | `case.tap` 之类，`case` 上没有这个成员 | 先 `app = case.init(case.config.<KEY>)`，设备动作写在 `app` 上 |
| `case_init` | `case.init` 的参数不是 `case.config.<KEY>` | 改成 `case.config.<KEY>` |
| `step_title` | `do` / `check` / `prep` 没有写文字标题 | 写这一步具体做什么的一句话 |
| `prep_missing` | `tests/` 里的用例没有 `prep` | 写 `case.nexus.prep("做了什么")`。确实不用做，标题写为什么，这句话会写入 Nexus |
| `prep_reason` | 前置标题只有「无」这类空话 | 写清做了什么；确实不用做，就写为什么 |
| `config_missing` | 脚本用了 `config.yaml` 里没有的应用，或没有 `config.yaml` | 补 `config.yaml`，参见 `repo-config` |
| `config_invalid` | `config.yaml` 格式不对 | 顶层 `apps`，每个应用要有 `platform` 和 `app_id`，key 以英文字母开头 |
| `page_layout` | 页面文件位置不对 | 放到 `pages/<APP_KEY>/` 下，目录名等于 `config.yaml` 里的 key |

注意：

- 语法错误不在这里检查（解析不了的文件会被跳过），语法你自己能发现；
- 这个命令只看源码，**不检查方法名对不对**（变量名不固定）。写错 `app.xxx`，运行时会报出相近的名字，到时按提示改；
- 没提示不代表脚本能通过：控件在页面里是否存在、锚点是否准确，只有真正执行才知道；
- 自检不核对 `locator_spec` 是否来自真机。格式能解析不代表能点中。`locator_spec` 只从 `case_run` 之后 `capability_id=hierarchy` 的 `nodes` 或 `hierarchy_text` 里抄；没有这次运行的界面结构就不要写。
- 命令不存在说明本机没装 Scout，参见 `overview`。
