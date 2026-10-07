---
title:
aliases:
tags:
  - windows
description:
draft: false
---
## Forward Slash `/`

Unix, Linux, web pages, URLs, and HTML all use the **forward slash `/`**.

```
https://www.baidu.com/img/xxx.png
```

## Backslash `\`

Windows File Explorer (This PC) uses the **backslash `\`** to separate folders. Example:

```
C:\Users\windows\Pictures
```

This is an old tradition set by Microsoft, used **only for local file paths in the Windows operating system**.

Historical reason: Early Windows 3.x was just a graphical shell running on top of DOS (Microsoft's early system), similar to a desktop environment in Linux. DOS originally had no directory feature, and `/` was already used for command arguments. To avoid symbol conflicts, when adding the directory feature, Microsoft chose `\` as the directory separator. Later, Microsoft developed the independent NT [[单词#Kernel 内核|Kernel]] (XP, Win7, Win10/11), which no longer depends on DOS, but kept the `\` habit for backward compatibility with old software. Modern Windows actually also supports writing paths with forward slashes `/`; it's just that the system displays backslashes by default.
