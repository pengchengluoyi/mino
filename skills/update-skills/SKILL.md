---
name: update-skills
description: 更新已安装的 Mino 技能，以及在使用技能时发现说明有误、缺内容、与实际不符时怎么反馈。用户说“更新技能”“技能过期了”，或你按技能操作时发现它和实际情况对不上时使用。
---

# 更新技能与反馈问题

技能不是一次写完的：Mino 的 Scout、Nexus、Workbench 在变，技能也跟着变。源地址是 `https://github.com/pengchengluoyi/mino.git`，技能目录在仓库的 `skills/` 下。安装和更新都按**你所在 agent 惯用的装 Skill 方式**做。

## 怎么知道装的是什么版本

以源仓库根目录的 `VERSION` 和 `CHANGELOG.md` 为准（可从 GitHub 读取，或看你这次安装时带下来的副本）。`CHANGELOG.md` 每个版本会写**需要的 Scout / Nexus 版本**。技能说的和你实际遇到的对不上，先看是不是版本不匹配：

```bash
mino-scout --version      # 如果没有此参数，就读 mino_scout/VERSION 或用 mino-scout status
```

## 更新

技能目录必须是 `https://github.com/pengchengluoyi/mino.git` 的检出，或指向该检出 `skills/<名字>` 的符号链接。**不要把技能文件复制到 agent 的技能目录。**

已有检出时：

```bash
git -C <mino检出> pull --ff-only
```

还没有检出时：

```bash
git clone https://github.com/pengchengluoyi/mino.git <目录>
```

然后把 agent 要加载的每个技能指到 `<目录>/skills/<技能名>`。之后只 `git pull`。

更新后告诉用户 `CHANGELOG.md` 里这次变了什么，并说明**需要新开一个会话**，新内容才会被加载。

用户只是要更新 Mino 技能时，只拉取这个仓库，不要改用户的业务仓库，也不要复制技能文件。

## 发现技能有问题怎么办

按技能操作时遇到下面情况，说明技能需要更新：说明与实际工具 / 页面 / 返回字段不一致，缺少你用到的步骤，或按它做会出错。

1. **先把眼前的任务做完**：按实际情况处理，不要因为技能不对就停手，也不要照着错的说明硬做；
2. **不要自己改已安装的技能文件**（安装结果会被下次更新覆盖，也会让别的会话行为不一致）；
3. 整理成一条反馈：**哪个技能、哪一段、说明写的是什么、实际是什么、证据**（报错原文、返回字段、截图描述），不含 token、密码、账号和内部地址；
4. 把反馈交给用户，由用户决定提到哪里。用户同意并且本机有 `gh` 时，可以在 `pengchengluoyi/mino` 提 issue：

```bash
gh issue create --repo pengchengluoyi/mino --title "<技能名>: <一句话>" --body "<上面整理的内容>"
```

未经用户同意不要提交 issue，这是对外发布内容。

## 给维护技能的人

- 改了技能：同时更新 `VERSION` 和 `CHANGELOG.md`（写清这个版本需要的 Scout / Nexus 版本）；
- 一个技能只做一件事，新增能力优先新建技能，而不是往已有技能里塞；
- 技能里不要手抄会变化的清单（方法列表、字段表），写“以本机安装的运行时 / 接口返回为准”并指明去哪读；
- 描述（`description`）要写清**什么时候用**，agent 靠它决定加载哪个技能；
- 安装说明只给源地址，让各 agent 按自己的方式装。
