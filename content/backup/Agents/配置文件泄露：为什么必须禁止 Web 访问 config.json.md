---
title: "配置文件泄露：为什么必须禁止 Web 访问 config.json"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Config-JSON-Leak-Why-Config-Files-Must-Be-Blocked-from-the-Web]]

# 一次 config.json 泄露：为什么站点的配置文件必须禁止 Web 访问

## 现象

一个单文件 PHP 站点把配置存在入口脚本旁边的 `config.json` 里。某天做例行安全检查时发现：

```
$ curl -s https://example.com/config.json
{
    "webdav_pass": "test",
    "minio_secret": "bnRPEmtSC8Z4E8NK",
    "password_hash": "$2y$10$...",
    ...
}
```

HTTP 200，整个文件内容都下来了。所有秘密——对象存储的密钥、WebDAV 密码、管理员密码哈希——都能公开下载。站点本身一切正常，之所以没人发现，是因为应用自己从来不链接到这个文件。

## 原理

Nginx（以及大多数 Web 服务器）会把请求映射到的**任何存在的静态文件**直接发出去，除非有规则显式拦掉：

- PHP 入口脚本是走 `location` 块路由的，但 `config.json` 不是 PHP，它命中**默认的静态文件处理**（`try_files $uri $uri/ ...`），被原样返回。
- 站点一般会维护一份「敏感文件」黑名单正则（`\.env`、`.git`、`.bak`、README…），但这份清单是手工维护的，**很容易漏掉应用自己的文件**，比如 `config.json`、`settings.json`、`db.json`。
- 单文件应用是最糟的情况：写着明文密钥的配置就躺在 **Web 根目录里**，紧挨着代码。

雪上加霜的两点：

1. 密码是 bcrypt 哈希（`password_hash()`），所以不能直接反推——但能下载到的哈希就**可以离线暴力破解**。明文字段（S3 密钥、WebDAV 密码）则是当场泄露。
2. 改权限（`600`、属主 `www`）没用：Web 服务器自己就是以 `www` 身份运行的，它读得到，也照发不误。

## 检测

每个站点都该过的五秒检查：

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://example.com/config.json
```

返回 200 就是有泄露。再顺手探几个变体：`?v=1`、`../config.json`，以及其他应用文件（`composer.json` 通常是被拦的，但 `config.php`、`config.yaml`、`*.tar.gz` 备份呢？）。

## 修复

在 nginx 的敏感文件黑名单里加进去：

```nginx
location ~* (\.user\.ini|config\.json|\.env.*|\.bak(up)?|...) $ {
    return 404;
}
```

然后 reload 并验证，包括绕过变体一起验：

```bash
nginx -t && nginx -s reload
curl -s -o /dev/null -w "%{http_code}\n" https://example.com/config.json        # 404
curl -s -o /dev/null -w "%{http_code}\n" "https://example.com/config.json?v=1"  # 404
```

## 纵深防御

- **把配置移出 Web 根目录**（比如 `/etc/myapp/config.json`，或者文档根目录的上一级），用绝对路径引用。这才是真正的修法——靠 nginx 黑名单是打地鼠。
- **敏感信息一律哈希**——密码用 `password_hash()`，能不存明文 API 密钥就别存（如果存在过泄露窗口，至少要轮换）。
- **保持 `600` 权限 + 专用服务账号**（挡不住 Web 服务器，但能挡住本机其他用户）。
- **有泄露窗口就轮换密钥**——只要这个文件曾经可以被下载一段时间，就当它已经被下载了。换掉 S3 密钥、WebDAV 密码、管理员密码。
- **把那条 curl 探测写进部署检查清单**——泄露是无声的，只有主动查才会发现。

## 小结

- Nginx 默认会提供存在的静态文件；你的 config.json 是静态文件，不拦就会被发出去。
- 敏感文件黑名单靠手工维护，总会漏——要针对应用自己的文件做审计（`config.json`、`*.tar.gz`、备份）。
- 检测只要一条 curl，修只要一条 nginx 规则；而稳妥的修法是把配置整个挪出 Web 根目录。
