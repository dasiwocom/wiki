---
title: Obsidian 与 Quartz 的协作工作流
date:
  "{ date:YYYY-MM-DD }":
tags:
  - obsidian
  - quartz
draft: false
authors:
  - 小古
class: "[[Obsidian/index|index]]"
---

# Obsidian 与 Quartz 的协作工作流


# Obsidian 与 Quartz 的协作工作流

> 我在 Obsidian 里写，Quartz 负责把它变成网站。两边各管一段，搞清楚边界就不会互相干扰。

## 仓库结构

```
D:\everything\projects\wiki\
├── content\          ← 这个目录就是 Obsidian 仓库的根（vault）
│   ├── index.md      ← 网站首页
│   ├── Obsidian\     ← 主题文件夹，一个文件夹一个主题
│   ├── templates\    ← 笔记模板
│   └── Attachments\  ← 图片等附件
├── quartz.config.default.yaml   ← Quartz 配置
└── .github\workflows\           ← 自动部署
```

**关键**：`content\` 既是 Obsidian 仓库，又是网站的源目录。在 Obsidian 里新建文件 = 给网站加页面，不需要任何同步操作。

## 从写完到上线

1. 在 Obsidian 里写笔记，保存
2. `git add .` → `git commit -m "说明"` → `git push`
3. push 到 **`v5` 分支**会触发 `.github/workflows/deploy.yaml`，自动构建并发布

中间不需要手动导出、手动上传。

## 哪些东西不会发到网站上

| 内容 | 被谁拦下 |
| --- | --- |
| `node_modules`、`public` | `.gitignore` |
| `.obsidian\`（Obsidian 自己的配置） | `.gitignore` |
| `templates\`、`private\` | Quartz 配置的 `ignorePatterns` |
| frontmatter 写了 `draft: true` 的笔记 | Quartz 构建时跳过 |

所以**本地配置和草稿不会泄露到公网**，可以放心折腾。

## Quartz 会自动生成的东西

- **文件夹页**：访问 `wiki.dasiwo.com/Linux` 会看到该文件夹下所有笔记的列表。这个功能由 `@quartz-community/folder-page` 插件提供，文件夹里**没有 index.md 也会自动生成**虚拟页。
- **左侧文件树**：`explorer` 插件，整站导航。

手写 `index.md` 的作用是**顶替**那个虚拟页，在自动列表上方加自己的导读。文件夹空着的时候没必要写。

## 两边的认知差异（容易踩坑）

| 在 Obsidian 里 | 在 Quartz 网站上 |
| --- | --- |
| `[[Linux]]` 指向一篇叫 Linux 的笔记，没有就是死链（点击会新建） | `[[Linux]]` 会解析到 Linux 文件夹的文件夹页，**是有效链接** |
| 模板变量 `{{title}}`、`{{date}}` 在插入时展开 | 网站看到的是展开后的结果 |

换句话说：**同一个双链，在两边意义不同**。在 Obsidian 里点文件夹链接会误建笔记，注意别手滑。
