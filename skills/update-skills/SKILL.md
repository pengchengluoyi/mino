---
name: update-skills
description: 更新已安装的 Mino 技能，以及在使用技能时发现说明有误、缺内容、与实际不符时怎么反馈。用户说“更新技能”“技能过期了”，或你按技能操作时发现它和实际情况对不上时使用。
---

# 更新技能与反馈问题

技能不是一次写完的：Mino 的 Scout、Nexus、Workbench 在变，技能也跟着变。技能放在 `https://github.com/pengchengluoyi/mino.git`，用 git 更新。

## 怎么知道装的是什么版本

技能仓库的根目录有 `VERSION` 和 `CHANGELOG.md`。克隆目录默认是 `~/.mino/skills-src`：

```bash
cat ~/.mino/skills-src/VERSION
git -C ~/.mino/skills-src log --oneline -5
```

`CHANGELOG.md` 每个版本会写**需要的 Scout / Nexus 版本**。技能说的和你实际遇到的对不上，先看是不是版本不匹配：

```bash
mino-scout --version      # 如果没有此参数，就读 mino_scout/VERSION 或用 mino-scout status
```

## 更新

```bash
git -C ~/.mino/skills-src pull --ff-only
cat ~/.mino/skills-src/CHANGELOG.md | head -40
```

- 技能是按软链接安装的（推荐，`ls -l ~/.claude/skills` 里每项都指向 `skills-src`），`pull` 完就是最新，**不用再复制**；
- 是复制安装的，把 `skills-src/skills/` 下的目录重新复制过去覆盖；
- `pull` 提示本地有改动或不能快进：停下告诉用户，**不要 `reset --hard`、不要 force**；
- 更新后告诉用户 `CHANGELOG` 里这次变了什么，并说明**需要新开一个会话**，新内容才会被加载。

## 发现技能有问题怎么办

按技能操作时遇到下面情况，说明技能需要更新：说明与实际工具 / 页面 / 返回字段不一致，缺少你用到的步骤，或按它做会出错。

1. **先把眼前的任务做完**：按实际情况处理，不要因为技能不对就停手，也不要照着错的说明硬做；
2. **不要自己改已安装的技能文件**（安装目录是共享的，改了会被下次更新覆盖，也会让别的会话行为不一致）；
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
- 描述（`description`）要写清**什么时候用**，agent 靠它决定加载哪个技能。
