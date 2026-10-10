---
title: 第二大脑
aliases: [首页, Home]
tags: [MOC]
description: 一个公开的学习笔记库：Windows 技巧、Markdown、HTML、51 单片机、英语词汇
created: 2026-10-10
draft: false
---

# 第二大脑

这里放我学过的东西。

写得比较随意——遇到什么记什么，搞懂了就展开写细，没搞懂就标着待查证。

## 区域

| 区域 | 内容 |
|---|---|
| 笔记 | Windows 技巧、Markdown、HTML、51 单片机 |
| 手册 | 这个库自己的规则、结构与维护方式 |
| 词库 | 英语单词与缩写 |
| 提示词 | 复用prompt |
| 日记 | 日常记录 |
| 项目 | 正在推进的事 |
| 领域 | 长期在管的事 |
| 资料 | 参考资料 |

## 笔记分布

```mermaid
graph TD
    Root[笔记] --> A[Windows]
    Root --> B[Markdown]
    Root --> C[HTML]
    Root --> D[51单片机]

    A --> A1[Windows 回收站]
    A --> A2[shell 命令速查]
    A --> A3[正斜杠与反斜杠]

    B --> B1[标准语法]
    B --> B2[扩展语法]

    D --> D1[Keil快捷键]
    D --> D2[数码管引脚定义]
    D --> D3[模块化编程]

    style Root fill:#4b6cb7,color:#fff
```

## 从哪开始

- 想查 Windows 上的操作：[[Windows 回收站]] · [[shell 命令速查]]
- 想写 Markdown：[[Obsidian 语法速查]]
- 在折腾单片机：[[Keil快捷键]] · [[单片机引脚接错程序正确不亮灯]]
- 想系统学 HTML：[[相对路径和绝对路径]] · [[URL 与 URL编码]]
