---
title: Quartz 部署 - Agent 操作参考提示词
---

# Quartz 部署到 GitHub Pages - Agent 操作参考提示词

> 通用参考模板。把本文件交给任意 AI Agent，让它按实际环境适配执行。

## 任务

用 Quartz 5 把一个 Obsidian 笔记库构建成静态网站，部署到 GitHub Pages，交付可访问的网址。

## 前置条件（按实际环境确认）

- 电脑能运行 Node.js（`npm` / `npx` 可用）
- Git 已安装；推荐用 SSH 连接 GitHub（`ssh -T git@github.com` 能返回 `Hi 用户名` 即免密可用）
- 有 GitHub 账号；知道笔记源目录在哪

## 流程（从零到上线）

### 1. 建仓库

GitHub 新建仓库，**Repository template 选 `jackyzha0/quartz`**，仓库名如 `wiki`。

```bash
git clone git@github.com:用户名/仓库名.git
cd 仓库名
```

记下当前分支名（模板默认可能是 `v5`，后续工作流要用）。

### 2. 放笔记

把 Obsidian 笔记复制进 `content/`（跳过 `.obsidian` 等配置目录）。

**注意**：如果源库里有超过 100 MB 的大文件（如 PDF 电子书），**不建议**放进 content——GitHub 单文件上限 100 MB，放了会导致 push 失败或极慢。

没有首页就复制一篇介绍笔记为 `content/index.md`，顶部加：

```markdown
---
title: 站点名
---
```

### 3. 本地预览（可选，推荐先看效果）

```bash
npx quartz build --serve
```

浏览器打开 `http://localhost:8080`，停止按 `Ctrl + C`。

### 4. 配置（重要）

改 `quartz.config.default.yaml` 里的 `baseUrl`：

- 无自定义域名：`用户名.github.io/仓库名`
- 有自定义域名：`你的域名`（如 `wiki.example.com`，不含 https:// 和尾斜杠）

`pageTitle` 顺手改成站点名。**不改 baseUrl 会导致全站链接路径错误**（丢前缀、指向模板官方域名）。

### 5. 启用 Pages（先做再推）

仓库 → **Settings** → **Pages** → Source 选 **GitHub Actions** → Save。

不先做的话，部署工作流会秒失败。页面上"GitHub Pages Jekyll / Static HTML"建议按钮不用点。

### 6. 新建部署工作流

**模板自带的工作流是部署到 Cloudflare 的（且仅限官方仓库运行），别用。** 新建 `.github/workflows/deploy.yaml`，代码见文末，分支名改成第 1 步记下的。

### 7. 推送

新仓库首次先配身份：

```bash
git config user.name "你的名字"
git config user.email "你的邮箱"
```

```bash
git add .
git commit -m "first deploy"
git push
```

### 8. 验收

Actions 里「Deploy Quartz site to GitHub Pages」变绿后（2–3 分钟），访问：

```
https://用户名.github.io/仓库名/
```

页面能正常渲染笔记内容即成功。

### 9. 自定义域名（可选）

1. 域名服务商加 DNS：`CNAME` 子域名 → `用户名.github.io`
2. Pages → Custom domain 填**完整域名** → Save
3. `baseUrl` 同步改成新域名，重新 push
4. 等 DNS check + HTTPS 证书（最长约 1 小时；报 `InvalidDNSError` 多半是传播延迟，记录没错就等一会儿再点 Check again）
5. 证书可用后勾 Enforce HTTPS

**注意**：设置自定义域名后，访问地址变成 `域名/`，**不再带 `/仓库名`**。

### 10. 日常更新

```bash
git add .
git commit -m "改了什么"
git push
```

等 Actions 变绿自动上线。日常只动 `content/`。

## 常见坑（参考案例，遇到对照处理）

| 现象 | 原因 | 处理 |
|---|---|---|
| push 卡住且体积很大 / 报 100 MB 限制 | content 混入了大文件 | 删掉大文件；`git reset HEAD~1` 撤销错误提交；重新 `git add .`、commit；`git push --force`（单人仓库 force 安全） |
| 部署工作流 1 秒结束 | Pages 没启用 / 用了模板自带工作流 / 分支不匹配 | 检查第 5、6 步 |
| commit 报 Author identity unknown | 没配 git 身份 | 先 `git config user.name` / `user.email` |
| commit 后黑屏 | 漏写 `-m` 进了 vim | 按 `Esc` 输入 `:wq` 退出；之后记得带 `-m` |
| 访问 Site not found / 404 | 构建没完成，或路径多/少了 `/仓库名` | 等 Actions 变绿；无域名带 `/仓库名`，有域名不带 |
| 点目录/搜索 404 且路径缺前缀 | baseUrl 错误 | 修正 baseUrl 重新 push |
| Custom domain 报 not properly formatted | 输入框填了路径而非域名 | 填完整域名或留空 |
| public 里有奇怪文件 | 旧构建残留 | 直接删 public 文件夹，下次构建自动重建（public 被 git 忽略） |

## deploy.yaml（复制即用，分支名按实际改）

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v5
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    name: Build
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install dependencies
        run: npm install
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```
