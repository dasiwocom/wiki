---
title:
aliases:
tags:
  - php
description: 九个超级全局变量分别装着什么，$_SERVER 里几个常用键，以及怎么用 include 和 require 把代码拆成几个文件。
---

# 超级全局变量和 include，把代码拆成几个文件

## 超级全局变量：哪里都能用

前面说过函数里面看不到外面的变量。有一类变量是例外——**在任何地方、包括函数里面，都能直接读**。它们叫超级全局变量，一共九个。

| 变量 | 装什么 |
| --- | --- |
| `$_SERVER` | 这次请求的各种信息（网址、方法、客户端 IP） |
| `$_GET` | 网址问号后面的参数 |
| `$_POST` | 表单以 POST 方式提交的数据 |
| `$_REQUEST` | `GET` + `POST` + `COOKIE` 混在一起（**不要用**） |
| `$_FILES` | 上传的文件 |
| `$_COOKIE` | 浏览器带过来的 Cookie |
| `$_SESSION` | 服务器上属于这个用户的会话数据 |
| `$_ENV` | 环境变量 |
| `$GLOBALS` | 当前所有全局变量 |

它们都是数组。第 12～16 篇会分别用到 `$_GET`、`$_POST`、`$_FILES`、`$_COOKIE`、`$_SESSION`。这一篇先把最基础的 `$_SERVER` 和拆分代码的写法讲清楚。

## `$_SERVER` 里常用的几个

```php
<?php
echo $_SERVER["REQUEST_METHOD"];   // GET 或 POST
echo $_SERVER["SCRIPT_NAME"];      // 当前脚本的路径，比如 /user/list.php
echo $_SERVER["QUERY_STRING"];     // 问号后面的部分，比如 id=3&page=2
echo $_SERVER["HTTP_HOST"];        // 域名，localhost:8000
echo $_SERVER["REMOTE_ADDR"];      // 访客的 IP
echo $_SERVER["HTTP_USER_AGENT"];  // 浏览器标识
```

最常见的用途是**判断这是不是一次表单提交**：

```php
<?php
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    // 处理提交
}
```

**只能用 `===` 比，而且一定是大写的 `"POST"`。** 写成 `"post"` 永远不成立。

另一个常用的是 `PHP_SELF`（当前脚本的地址，用来让表单提交给自己），但它有个坑，见第 12 篇。

## 把代码拆成几个文件

一个网站每个页面都有相同的头部和底部。复制到十几个文件里，改一次标题要改十几处。

拆出去，用 `include` 引进来：

```php
<?php
// header.php
?><!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title><?php echo $pageTitle ?? "我的小站"; ?></title>
</head>
<body>
<nav>
  <a href="/index.php">首页</a>
  <a href="/about.php">关于</a>
</nav>
```

```php
<?php
// index.php
$pageTitle = "首页";
include __DIR__ . "/header.php";
?>
<main>
  <h1>欢迎</h1>
</body>
</html>
```

`include` 就相当于把那个文件的内容**原地粘贴进来**执行。所以 `header.php` 里能用到 `$pageTitle`——它和引用它的文件共享同一个作用域。

### 四个 include 的区别

| 写法 | 找不到文件时 |
| --- | --- |
| `include` | 警告，**继续往下跑** |
| `require` | **致命错误，立刻停** |
| `include_once` | 同 `include`，但只引一次 |
| `require_once` | 同 `require`，但只引一次 |

**需要的东西用 `require_once`**（比如函数库、数据库配置），缺了就没法运行。
**可选的、缺了也能凑合的用 `include`**（比如一段广告位）。

"once" 是防重复：同一个文件被引两次，里面的函数会重复定义，直接报错。

### 路径一定要用 `__DIR__`

```php
<?php
require_once __DIR__ . "/config.php";
require_once __DIR__ . "/lib/functions.php";
```

`__DIR__` 是当前文件所在的绝对目录（第 5 篇讲过）。用它拼出来的路径**不管你从哪个目录访问这个网站，都能找到文件**。

写成 `require "config.php"` 的话，它是相对"当前工作目录"找的——同一份代码换个运行方式就找不到了，这是新手最常见的报错之一。

### 一个典型的小站结构

```text
/var/www/site/
├── index.php
├── about.php
├── config.php              配置：数据库密码、站点名
├── lib/
│   └── functions.php       公共函数
├── templates/
│   ├── header.php
│   └── footer.php
└── uploads/                上传的文件
```

`config.php` 大概长这样：

```php
<?php
const DB_HOST = "localhost";
const DB_NAME = "mysite";
const DB_USER = "root";
const DB_PASS = "";

const SITE_NAME = "我的小站";
const PAGE_SIZE = 10;
const UPLOAD_DIR = __DIR__ . "/uploads";
```

`functions.php` 里放公共函数：

```php
<?php
function e(string $s): string {
    return htmlspecialchars($s, ENT_QUOTES, "UTF-8");
}

function redirect(string $url): void {
    header("Location: " . $url);
    exit;
}
```

那个 `e()` 是个很实用的习惯——**名字短，写输出转义的时候不会嫌麻烦**（"e" 就是 escape）。这样写模板的时候满篇是 `<?php echo e($user["name"]); ?>`，比每次都敲一遍 `htmlspecialchars` 轻松，也不容易漏。

`redirect()` 里的 `exit` 不能省，否则后面代码还会执行一遍。

## 用 `header()` 做页面跳转

```php
<?php
header("Location: /list.php?id=3");
exit;
```

两件事必须记住：

**① `header()` 之前不能有任何输出。** 一个空格、一个 BOM、甚至 `?>` 后面的一个换行都算输出，会导致"headers already sent"警告，跳转失效。这就是第 2 篇说的"纯 PHP 文件不写结尾 `?>`"的原因。

**② 跳转之后立刻 `exit`。** 不加的话，页面还没跳过去，后面的代码照样执行完了——如果后面是"删除数据"，那用户点一次会执行两次。

## 局部变量不会互相污染

同一个文件被不同页面 `include`，里面的变量是**各自独立的**——因为每次请求都是一个新的进程，跑完就全部销毁。所以不用担心 A 页面的 `$user` 影响到 B 页面。

但同一个页面里**用过的变量名不要重复**。`header.php` 里如果也用 `$list`，就会把你自己的数据冲掉。起名的时候带点前缀（`$navItems`、`$userList`）能避免这事。

## 一个例子：全站共用的头部和底部

```php
<?php
// header.php
require_once __DIR__ . "/functions.php";
?><!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title><?php echo e($pageTitle ?? SITE_NAME); ?> - <?php echo e(SITE_NAME); ?></title>
</head>
<body>
<header><h1><?php echo e(SITE_NAME); ?></h1></header>
```

```php
<?php
// footer.php
?>
<footer>
  <p><?php echo date("Y"); ?> · <?php echo e(SITE_NAME); ?></p>
</footer>
</body>
</html>
```

```php
<?php
// index.php
require_once __DIR__ . "/config.php";
$pageTitle = "首页";

require __DIR__ . "/header.php";
?>
<main>
  <p>现在是 <?php echo date("Y-m-d H:i:s"); ?></p>
</main>
<?php require __DIR__ . "/footer.php"; ?>
```

三个文件拼出来的页面上是完整的 HTML，标题是"首页 - 我的小站"，底部带着今年的年份。

延伸阅读：[PHP 超级全局变量 - 菜鸟教程](https://www.runoob.com/php/php-superglobals.html)，[PHP 包含文件](https://www.runoob.com/php/php-includes.html)，[PHP 官方手册：预定义变量](https://www.php.net/manual/zh/reserved.variables.php)。

---

> 系列第 11 / 19 篇 · ← 上一步：[[10-函数，参数、返回值、作用域和引用|函数]] · 下一步：[[12-接收表单数据，GET 和 POST|接收表单数据]] → · 路线图：[[PHP 总览]]
