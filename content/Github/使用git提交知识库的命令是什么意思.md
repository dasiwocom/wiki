
Git 这三步是所有版本控制的日常主流程，用一张图就能看懂：
顺带确认一下你的 wiki 项目现在的 git 状态：
你的项目**已经是 git 仓库了**，而且有 `.github` 目录，说明 push 大概率会自动触发部署。我确认
一下：
Git 就是给文件**拍存档**的系统——每 commit 一次就留一个版本，写崩了能一键回到上一版。三条命令对应装车、封箱、发货：

**`git add .`** —— 把改动收进"待提交清单"
那个 `.` 是"当前目录下所有文件"。你在 Obsidian 里写了 3 篇笔记，这一步就是把它们挑出来准备存档。想只挑一部分，就把 `.` 换成具体文件名（比如 `git add content/C语言`）。

**`git commit -m "new"`** —— 正式存档，并写句说明
`-m` 是 message 的缩写，后面引号里就是这次存档的说明文字。你写的 `"new"` 就是说明，随便写什么都行——不过以后翻历史时你会后悔，建议写具体点：`git commit -m "新增指针笔记和 Linux 常用命令"`。这一步做完，存档就存在你电脑上了，**GitHub 上还看不到**。

**`git push`** —— 把本地存档上传到 GitHub
只有这一步才联网。传完 GitHub 上就是最新的了。

为什么要分三步而不是一步？因为 `add` 让你**挑**——你可以只把写好的三篇加进去，把那篇半成品留在本地不提交。

---

**你的 wiki 项目现在的实际状况**（我刚查了）：

- `D:\everything\projects\wiki` **已经是 git 仓库**（有 `.git`、` .gitignore`）
- `.github/workflows/deploy.yaml` 会在你 **push 到 `v5` 分支**时自动构建并发布到 GitHub Pages
- 但其他几个 workflow（ci、docker 等）都带 `if: github.repository == 'jackyzha0/quartz'` 的判断——那是 Quartz 官方仓库专用的，你的仓库不会触发，忽略即可
- ⚠️ 你的 `baseUrl` 配的是 `wiki.dasiwo.com`，而 workflow 默认发到 GitHub Pages（`xxx.github.io`）。**想用你自己的域名还得配 CNAME**，这是后面要解决的事

所以以后你写完笔记想发布，标准四连：

```
git status          ← 先看看自己改了哪些文件（养成习惯）
git add .
git commit -m "写了什么"
git push
```

`git status` 是我强烈建议每次先跑的——它会告诉你哪些文件被改了、哪些还没 add，避免你误把不该传的东西传上去。要我现在帮你跑一下 `git status`，看看今天这些改动是啥状态吗？