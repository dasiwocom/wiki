---
title: 使用git提交笔记改动的命令是什么是什么意思
date: 2026-09-27
tags:
  - git
draft: false
authors:
  - 小古
---

# Git 三步：add、commit、push

> 写完笔记想发布到网站，就是这三条命令。理解它们各管一段，比死记顺序重要。

## 三个区域

Git 把文件流转分成三站，一条命令对应一站：

```
工作区（你刚改的文件）  --git add-->  暂存区（待提交清单）  --git commit-->  本地仓库（存档快照）  --git push-->  远程仓库（GitHub）
```

**只有最后一步联网。** commit 完的东西还在你自己电脑上，别人和网站都看不到。
## 逐条拆解

### `git add .`

把改动收进「待提交清单」。

- `.` = **当前所在目录**，不是固定的仓库根目录。终端停在 `D:\everything\projects\wiki` 时，它就代表整个仓库；停在 `content\` 里，就只收 content 的改动。建议每次站在仓库根目录执行。
- 它只收**有变化**的文件（新增 / 修改 / 删除），没动过的不会重复提交。
- `.gitignore` 里列的东西会被自动跳过——本仓库已排除 `node_modules`、`public`、`.DS_Store`。这些不该进仓库：`node_modules` 几百 MB 且能靠 `npm install` 重建，`public` 是构建产物。
- 想只提交一部分，把 `.` 换成具体路径：`git add content/C语言`。

### `git commit -m "说明文字"`

正式存档，`-m` 后面引号里是这次存档的说明（message）。

- 说明**一定要写具体**。`git commit -m "new"` 这种，三个月后翻历史等于没写。
- 好的写法：`git commit -m "新增指针笔记和 Linux 常用命令"`。
- 这一步是在本地留下一个可回退的版本节点，写崩了能退回来。

### `git push`

把本地存档上传到 GitHub，是唯一连网的步骤。

## 配套命令（比三条主线更常用）

| 命令 | 作用 |
| --- | --- |
| `git status` | 看当前状态：**红色**=已改未 add，**绿色**=已 add 待 commit。**每次提交前先跑它** |
| `git log --oneline` | 看历史存档，一行一个 |
| `git diff` | 看具体改了哪些内容（add 之前用） |

## 本仓库的自动化情况

- push 到 **`v5` 分支**会触发 `.github/workflows/deploy.yaml`，自动构建并发布到 GitHub Pages。
- 仓库里其他 workflow（ci、docker 等）都带 `if: github.repository == 'jackyzha0/quartz'` 判断，是 Quartz 官方仓库专用的，**对我们不生效**，不用管。

## 发布一次笔记的标准流程

```bash
git status                          # 先确认改了什么
git add .
git commit -m "做了什么改动"
git push
```




