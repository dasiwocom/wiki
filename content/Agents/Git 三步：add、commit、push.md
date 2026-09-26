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
- ⚠️ 待办：`baseUrl` 配的是 `wiki.dasiwo.com`，但 workflow 默认发到 `xxx.github.io`。要用自己的域名还需要配 CNAME。

## 发布一次笔记的标准流程

```bash
git status                          # 先确认改了什么
git add .
git commit -m "做了什么改动"
git push
```

<svg version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 248 400" width="248" height="400" class="excalidraw-svg"><!-- svg-source:excalidraw --><metadata></metadata><defs><style class="style-fonts">
      </style></defs><rect x="0" y="0" width="248" height="400" fill="#ffffff"></rect><g stroke-linecap="round" transform="translate(10 60) rotate(0 87 60)"><path d="M30 0 C59.73 3.08, 89.58 1.38, 144 0 M30 0 C57.61 -0.49, 87.81 -1, 144 0 M144 0 C162.97 -0.39, 173.1 9.35, 174 30 M144 0 C162.56 -0.4, 174.62 8.34, 174 30 M174 30 C175.53 51.06, 172.57 70.68, 174 90 M174 30 C175.26 46.44, 175.21 61.74, 174 90 M174 90 C174.42 108.89, 165.21 121.55, 144 120 M174 90 C173.3 110.33, 162.25 119.84, 144 120 M144 120 C118.46 121.6, 93.91 121.27, 30 120 M144 120 C111.04 121.74, 78.91 121.33, 30 120 M30 120 C11.9 119.85, -0.13 109.53, 0 90 M30 120 C8.14 121.61, 0.16 108.45, 0 90 M0 90 C-1.92 76.73, -0.12 59.77, 0 30 M0 90 C0.64 72.44, 0.55 55.36, 0 30 M0 30 C0 9.94, 11.74 -0.7, 30 0 M0 30 C1.44 10.88, 8.45 -0.3, 30 0" stroke="#1e1e1e" stroke-width="2" fill="none"></path></g><g stroke-linecap="round" transform="translate(167 10) rotate(0 35.5 190)"><path d="M17.75 0 C30.69 -0.83, 40.81 -1.73, 53.25 0 M17.75 0 C28.27 0.2, 39.27 0.8, 53.25 0 M53.25 0 C66.2 -0.26, 70.74 7.37, 71 17.75 M53.25 0 C64.57 2.09, 71.9 7.08, 71 17.75 M71 17.75 C70.52 152.78, 71.75 290.04, 71 362.25 M71 17.75 C70.18 97.32, 70.62 178.1, 71 362.25 M71 362.25 C71.29 373.89, 66.03 378.64, 53.25 380 M71 362.25 C73.13 373.74, 67 380.43, 53.25 380 M53.25 380 C41.91 381.56, 32.72 378.96, 17.75 380 M53.25 380 C43.93 380.16, 32.18 379.99, 17.75 380 M17.75 380 C6.14 381, -0.85 375.19, 0 362.25 M17.75 380 C5 381.23, -0.36 375.53, 0 362.25 M0 362.25 C-1.89 263.86, -1.2 167.74, 0 17.75 M0 362.25 C-0.31 264.13, 0.31 167.75, 0 17.75 M0 17.75 C-1.92 5.39, 5.04 0.01, 17.75 0 M0 17.75 C0.82 5.19, 7.15 -1.6, 17.75 0" stroke="#1e1e1e" stroke-width="2" fill="none"></path></g><g transform="translate(182.03955841064453 197.27833333333334) rotate(0 20.46044158935547 2.721666666666664)"><text x="20.46044158935547" y="3.9997591145833336" font-family="Helvetica, sans-serif, Segoe UI Emoji" font-size="4.733333333333333px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">Empty Web-Embed</text></g><g stroke-linecap="round" transform="translate(167 10) rotate(0 35.5 190) scale(1, 1)"><foreignObject style="width: 71px; height: 380px; border-width: medium; border-style: none; border-color: currentcolor; border-image: none;"><div xmlns="http://www.w3.org/1999/xhtml" style="width: 100%; height: 100%;"><iframe src="about:blank" allowfullscreen="" style="width: 100%; height: 100%; border-width: medium; border-style: none; border-color: currentcolor; border-image: none; border-radius: 17.75px; top: 0px; left: 0px;"></iframe></div></foreignObject></g></svg>



![[Pasted image 20260927013012.png]]