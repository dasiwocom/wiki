---
title: "排查 PHP 文件权限：编辑后站点报 Permission denied"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Debugging-PHP-File-Permissions-After-an-Edit]]

# 排查 PHP 文件权限：编辑之后出的问题

## 现象

你在服务器上用某个会**替换整个文件**的工具（patch、恢复备份、scp）编辑了一个 PHP 文件，突然整个站点开始返回 Fatal error —— 或者更糟，返回的是错误页面的缓存副本，看起来毫无变化。

## 根因

编辑器是以**当前用户**（root）的身份写文件的，把 PHP-FPM 的属主覆盖了。文件变成 `root:600`，FPM（以 `www` 身份运行）读不到，每个请求都死在 "Permission denied" 上。

## 检查清单

1. `ls -la file.php` —— 属主应该是 `www:www`，权限 `644`。
2. 修：`chown www:www file.php && chmod 644 file.php`。
3. PHP opcache 可能还握着旧代码一两秒 —— 等一下再验证。
4. 反向代理 / CDN 可能缓存了错误页 —— 用一个新的 URL 测试（`?v=timestamp`）。

## 相关

- [[配置文件泄露：为什么必须禁止 Web 访问 config.json]]
- [[Markdown 站点的服务端渲染]]
- [[为什么恢复备份后配置文件会 Permission Denied]]
