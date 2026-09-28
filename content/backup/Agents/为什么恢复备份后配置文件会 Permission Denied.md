---
title: "为什么恢复备份后配置文件会 Permission Denied"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Why-a-Config-File-Gets-Permission-Denied-After-Restoring-a-Backup]]

# 为什么恢复备份之后配置文件突然 Permission Denied

## 现象

在一台临时服务器上测试新版 PHP 面板，然后把生产环境的配置文件恢复回去，结果生产站点挂了，报：

```
Warning: file_get_contents(config.json): Failed to open stream: Permission denied
```

文件还在。大小看着也正常。但 PHP-FPM 读不了它了。

## 根因

恢复那一步是以 `root` 身份做的：

```bash
cp config.json /tmp/config.json.bak   # 备份（属主仍是 www）
# ...跑测试，测试过程改了 config.json ...
mv /tmp/config.json.bak config.json   # 以 root 身份恢复！
```

`mv` **不会保留目标文件原本的属主**。以 `root` 运行时，恢复出来的文件属主变成了 `root:root`。

而 PHP-FPM worker 是以 `www`（或 `www-data`）身份运行的。配置文件通常权限是 `600`（私有），所以 `www` 一点读权限都没有。结果：每个要加载配置的请求都死在 `Permission denied` 上 —— 哪怕路径和内容都是对的。

注意：这里用 `cp` 就是安全的（备份保留了 `www` 属主），恢复之后再 `chown www:www` 也能修好。陷阱在于**以 root 恢复会静默改变属主** —— 你只在应用读不到自己的配置时才会发现。

## 修法

```bash
chown www:www config.json
ls -la config.json   # 确认：www www
```

然后在告诉任何人「修好了」之前，重新验证一遍站点（首页、管理页、列表 API）。

## 教训

1. **备份和恢复必须保留属主。** 往返操作优先用 `cp`，或者在 `mv` 恢复之后总是跑一次 `chown`。
2. **任何文件恢复之后都检查 `ls -la`** —— 属主/属组/权限是文件状态的一部分，不只是内容。
3. **当 PHP-FPM 对一个存在的文件报 `Permission denied` 时**，首先怀疑属主：worker 用户（`www`）必须是属主（或者该文件必须组可读）。

## 相关：同类 bug

`write_file` 在 `www` 属主的 web 根目录里以 `root` 身份创建了 Markdown 笔记 → 站点 API 返回空内容。修法完全相同：写完之后 `chown www:www`。

## 相关：把 root 属主的私有文件挂进 PHP 站点

同样的症状，不同的原因：通过「自定义渲染路径」去渲染 web 根目录之外的一个目录（比如 `/root/.hermes/memories`）。API 返回了坏掉的 JSON（前端报 `null is not an object`），因为：

- 目录链 `/root` → `.hermes` → `memories` 对 `www` 是可进入的（权限 `705`），所以**文件列表**是好的。
- 但那些 Markdown 文件本身是 `600 root` → `file_get_contents()` 失败，抛出的 PHP `Warning` 被前置到了 JSON 正文里 → 前端 `r.json()` 抛异常 → null 错误。

修法：`chmod 644 <file>.md`（保留 `root` 属主，让 `www` 能读）。如果以后这些文件被重新写成私有权限，错误会回来 —— 任何重写之后都要留意权限检查。
