---
name: update-skills
description: 从 GitHub 安装或更新已装到 Cursor 的 Mino 技能，以及发现技能说明有误时怎么反馈。用户说“更新技能”“技能过期了”“重新安装 mino skill”，或你按技能操作时发现说明和实际不符时使用。
---

# 更新技能与反馈问题

源仓库：`https://github.com/pengchengluoyi/mino.git`（技能在 `skills/`）。版本以根目录 `VERSION`、`CHANGELOG.md` 为准。

## Cursor：禁止软链

**禁止**把 `~/.cursor/skills/<名>` 做成指向本机业务仓（如 `code/mino`）的符号链接。  
安装与更新都必须：**从 GitHub 拉一份 → 复制成真实目录/文件**。

## 安装 / 更新（Cursor）

在临时目录 clone（或 `--depth 1`），再拷进 Cursor 技能目录：

```bash
TMP=$(mktemp -d)
git clone --depth 1 https://github.com/pengchengluoyi/mino.git "$TMP/mino"
DEST="$HOME/.cursor/skills"
mkdir -p "$DEST"
# 先清掉旧的同名技能与版本文件（含误建的软链）
rm -rf "$DEST"/VERSION "$DEST"/CHANGELOG.md \
  "$DEST"/overview "$DEST"/onboarding "$DEST"/repo-config \
  "$DEST"/write-test-cases "$DEST"/check-test-cases \
  "$DEST"/run-test-cases "$DEST"/update-skills
cp "$TMP/mino/VERSION" "$DEST/VERSION"
cp "$TMP/mino/CHANGELOG.md" "$DEST/CHANGELOG.md"
for d in "$TMP/mino/skills"/*/ ; do
  [ -f "$d/SKILL.md" ] || continue
  cp -R "$d" "$DEST/$(basename "$d")"
done
rm -rf "$TMP"
# 确认没有软链
find "$DEST" -type l
cat "$DEST/VERSION"
```

其他 agent：按其惯用装 Skill 方式安装同一 GitHub 地址；**同样不要用指向随意本地仓的软链冒充已发布版本**。

更新后告诉用户 `CHANGELOG.md` 本版改了什么，并说明**需要新开一个会话**才会加载新内容。

只更新 Mino 技能时：只动上述技能目录，不要改用户业务仓库。

## 怎么知道装的是什么版本

看 **`$HOME/.cursor/skills/VERSION`**（以及同目录 `CHANGELOG.md`），不要看本机正在开发的 `code/mino`，除非用户明确说在维护技能源仓。

## 发现技能有问题怎么办

1. **先把眼前的任务做完**：按实际情况处理，不要硬照错说明；  
2. **不要直接改** `$HOME/.cursor/skills/` 里的已装文件来“修永久版”（下次更新会覆盖，也和发布源不一致）；  
3. 整理反馈：哪个技能、哪一段、写的是什么、实际是什么、证据（无 token/密码）；  
4. 交给用户；用户同意且有 `gh` 时可在 `pengchengluoyi/mino` 开 issue。

## 给维护技能的人

- 改技能：改 GitHub 源仓，更新 `VERSION` 与 `CHANGELOG.md`，push 后让使用者按上一节重装；  
- 一个技能只做一件事；  
- 不要手抄易变清单，写“以运行时 / 接口返回为准”；  
- `description` 写清何时使用。
