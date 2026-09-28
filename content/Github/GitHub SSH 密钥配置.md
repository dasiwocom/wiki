---
title: "GitHub SSH 密钥配置"
date: 2026-09-27
authors: []
tags: [github]
draft: false
---



> SSH key 管「这台电脑能不能推」，`user.name` 管「这条提交算谁的」。两套独立的东西，互不相干。

## 1、生成密钥（

每台电脑只做一次）

```
ssh-keygen -t ed25519 -C "你的邮箱"
```

一路回车即可，不需要设密码。

## 2、把公钥贴到 GitHub

```
cat ~/.ssh/id_ed25519.pub
```

复制输出的整行内容 → GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴 → Add。

## 3、验证

```
ssh -T git@github.com
```

注意 `git@github.com` 是一个整体，中间**没有空格**。看到 `Hi 用户名! You've successfully authenticated` 即成功，以后 push / pull 全程免密。

## 与 user.name 的区别

| | SSH key | user.name / user.email |
|---|---|---|
| 管什么 | 推送权限（这台电脑能不能动你的仓库） | 提交署名（这条 commit 写的是谁） |
| 参与验证吗 | 参与，配对失败推不上去 | 不参与，纯签名 |
| GitHub 怎么用 | 识别你的电脑 | 靠邮箱认领提交（头像、贡献绿格子） |
| 存在哪 | 电脑 `~/.ssh/` + GitHub 账号里 | 电脑 `~/.gitconfig` |

换新电脑：**两样都要重新弄**——先生成新密钥贴到 GitHub（本文），再 `git config --global` 报一次名字