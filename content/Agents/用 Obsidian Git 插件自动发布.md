---
title: "用 Obsidian Git 插件自动发布"
date: 2026-09-27
tags:
  - obsidian
  - git
draft: false
authors:
  - 小古
---

# 用 Obsidian Git 插件自动发布

> 每写完笔记都要敲 `git add` / `commit` / `push` 三行太麻烦。装 Obsidian Git 插件后，在 Obsidian 里点一下就行，还能定时自动提交。

插件地址：`Vinzent03/obsidian-git`（Obsidian 社区插件市场里搜 "Git"）

## 安装

1. Obsidian 设置 → **第三方插件** → 关掉「安全模式」
2. 点「浏览」→ 搜索 **Obsidian Git** → 安装 → 启用

## 推荐配置

设置 → Obsidian Git：

| 设置项 | 建议值 | 说明 |
| --- | --- | --- |
| Vault backup interval | `30` | 每 30 分钟自动 commit 一次；填 `0` 表示只用手动 |
| Commit message | `vault backup: {{date}}` | `{{date}}` 会自动替换成时间 |
| Push on backup | 开 | 提交后自动 push |
| Pull update on startup | 开 | 打开 Obsidian 时先拉最新，多设备用得上 |
| Disable on mobile | 开 | 手机端跑 git 容易出问题 |

**手动触发**（不想等自动的话）：`Ctrl+P` 打开命令面板，搜：
- `Obsidian Git: Create backup` —— 提交
- `Obsidian Git: Push` —— 推送到远程

左侧状态栏也会多一个 Git 图标，点它能看状态和快捷操作。

## ⚠️ 三个要注意的点

**1. 分支必须是 `v5`**

本仓库的部署工作流 `.github/workflows/deploy.yaml` 只在 **push 到 `v5` 分支**时触发。当前仓库分支就是 `v5`（`.git/HEAD` 里写的），正常情况不用管；但如果哪天切了分支，push 了网站也不会更新。

**2. 自动提交会频繁触发构建**

每 commit + push 一次，GitHub Actions 就重新构建一次网站。间隔设太短（比如 5 分钟）会产生一大堆无意义的构建记录和一堆 `vault backup` 提交。建议 **30 分钟以上，或者干脆关掉自动、写完手动点一下**。

**3. 插件解决不了认证问题**

它本质是把 git 命令包了一层。如果 push 报权限错误，说明 GitHub 认证没配好（Windows 上一般用 Git Credential Manager，或配 SSH key），得先在终端里让 `git push` 能跑通，插件才能用。

## 手机端

Obsidian 移动版的 git 支持很有限，建议按上面说的开启 **Disable on mobile**，手机上只写不发布，回到电脑再同步。
