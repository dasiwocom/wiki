---
title:
aliases:
tags:
  - php
description: PHP 文件的写法、注释、echo 和 var_dump 的区别，以及单引号、双引号和 heredoc 三种字符串到底差在哪。
---

# PHP 代码怎么写，输出、注释和字符串

## 一个 PHP 文件长什么样

最短的 PHP 文件：

```php
<?php
echo "你好";
```

`<?php` 是入口，告诉服务器"从这里开始是 PHP 代码"。字符串外面那对引号里的内容原样送到页面。

PHP 文件里可以既写 HTML 又写 PHP：

```php
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>问候</title>
</head>
<body>
  <h1>今天是 <?php echo date("Y年m月d日"); ?></h1>
  <p><?php echo "这段是 PHP 算出来的"; ?></p>
</body>
</html>
```

`<?php ... ?>` 之外的部分原样输出，之间的部分被当成代码执行。这就是"PHP 生成 HTML"最直观的样子。

**纯 PHP 文件（整个文件只有代码、不输出 HTML）建议不写结尾的 `?>`。** 原因很实际：结束标记后面的一个空格或一个换行都会被当成输出。如果这个文件是被 `include` 进来的，那点多余空白就会跑到页面最前面，可能让图片显示不出来、或者让"跳转"失败。

## 每条语句都要分号

```php
<?php
$name = "小明";
echo $name;
```

漏了分号会直接报语法错误，而且报错行号常常指向下一行，容易看错地方。

## 注释

```php
<?php
// 单行注释，最常用

# 单行注释，也可以用井号（不常见，看到别懵）

/*
  多行注释
  可以写好几行
*/

echo "注释不会输出";
```

```text
注释不会输出
```

## 输出的几个办法

### `echo` 和 `print`

```php
<?php
echo "你好";
echo "你好", "世界";        // echo 可以一次输出多个，用逗号隔开
print "也可以，但一次只能输出一个";
```

```text
你好你好世界也可以，但一次只能输出一个
```

`echo` 没有返回值，`print` 返回 `1`。日常用 `echo` 就够了，`print` 基本可以当不存在。

### `print_r`：给人看数组

```php
<?php
$user = ["name" => "小明", "age" => 18];
print_r($user);
```

```text
Array
(
    [name] => 小明
    [age] => 18
)
```

### `var_dump`：给调试看类型

```php
<?php
$user = ["name" => "小明", "age" => 18];
var_dump($user);
```

```text
array(2) {
  ["name"]=>
  string(6) "小明"
  ["age"]=>
  int(18)
}
```

`var_dump` 会连**类型和长度**一起打出来。字符串 `"18"` 和数字 `18` 用 `echo` 看一模一样，用 `var_dump` 一眼就分得清——哪一个才是排查问题的关键。

**这两个函数是 PHP 调试的主力。** 想知道某个变量到底是什么，`var_dump($变量);` 打在页面上看一眼，比盯着代码猜快得多。

### 输出时别忘了转义

```php
<?php
$comment = '<script>alert("你被坑了")</script>';
echo $comment;                              // 危险：这段脚本会真的执行
echo htmlspecialchars($comment);            // 安全：原样显示成文字
```

只要是**用户填进来的内容**，输出前一律过一遍 `htmlspecialchars`。否则别人在评论框里写一段 `<script>`，所有看这条评论的人都会中招。

## 三种字符串

### 单引号：几乎原样

```php
<?php
$name = "小明";
echo '你好，$name';        // 不解析变量
echo '换行不生效：\n';
```

```text
你好，$name换行不生效：\n
```

单引号里只认两个转义：`\'` 表示单引号本身，`\\` 表示一个反斜杠。其他都原样。

### 双引号：会解析变量和转义

```php
<?php
$name = "小明";
echo "你好，$name";
echo "换行在这里生效：\n";
echo "他叫 {$name}，今年 18";
```

```text
你好，小明
换行在这里生效：
他叫 小明，今年 18
```

双引号里 `$name` 会被替换成变量的值。**变量后面紧跟中文或其他字符时，用花括号包起来写 `{$name}`**，否则 PHP 可能把后面的字也当成变量名的一部分去解析。

常用的转义：

| 写法 | 意思 |
| --- | --- |
| `\n` | 换行 |
| `\t` | 制表符 |
| `\"` | 双引号 |
| `\\` | 反斜杠 |
| `\$` | 一个真的美元符号，不当变量 |

注意 `\n` 是"源码里的换行符"，在浏览器里显示的效果取决于你输出到哪：输出到 HTML 时它不换行（HTML 里换行要 `<br>`），输出到终端或文本文件时才换行。

### 该用哪个

**默认用单引号，里面要插变量时才换双引号。**

单引号不解析，PHP 不用去扫描字符串内容，也少一类意外。写 SQL、写正则、写文件路径的时候尤其要用单引号——里面经常有 `$` 和 `\`，双引号会把它们吃掉。

### heredoc 和 nowdoc：写多行

一段 HTML 或一段长文本，用引号拼起来很难看。用 heredoc：

```php
<?php
$name = "小明";

$html = <<<HTML
  <div class="card">
    <h2>你好，{$name}</h2>
    <p>这是 heredoc 写出来的</p>
  </div>
HTML;                        // 结束标记必须顶格，这一行除了它和分号不能有别的东西

echo $html;
```

`<<<HTML` 后面跟一个自定义的标识符，一直写到单独一行的 `HTML` 为止。它和双引号一样**会解析变量和转义**。

nowdoc 是不解析的版本，标识符外面加单引号：

```php
<?php
$tpl = <<<'TXT'
原样输出 $name，也不解析 \n
TXT;
```

一个容易踩的坑：**结束标记那一行必须顶格写，前面一个空格都不能有**，后面也只能跟分号。老版本的 PHP 更严格，现在允许后面跟其他内容，但顶格这条一直有效。写不对会直接报语法错误。

## 大小写

```php
<?php
$name = "小明";
echo $Name;         // 报错：Undefined variable
```

**变量名区分大小写**，`$name` 和 `$Name` 是两个东西。

函数名和关键字**不区分**（`echo`、`ECHO`、`Echo` 都行），但没人这么写，一律小写。

## 一个完整的例子

```php
<?php
$goods = [
  ["name" => "键盘", "price" => 199.5, "num" => 1],
  ["name" => "鼠标", "price" => 89.0, "num" => 2],
];

$total = 0;
foreach ($goods as $g) {
  $total += $g["price"] * $g["num"];
}
?>
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>购物清单</title>
</head>
<body>
  <h1>购物清单</h1>
  <ul>
    <?php foreach ($goods as $g): ?>
      <li><?php echo htmlspecialchars($g["name"]); ?> × <?php echo $g["num"]; ?></li>
    <?php endforeach; ?>
  </ul>
  <p>合计：<?php echo $total; ?> 元</p>
</body>
</html>
```

页面上的结果：

```text
购物清单

键盘 × 1
鼠标 × 2

合计：377.5 元
```

最后的 `<?php endforeach; ?>` 是 `foreach` 的另一种写法（叫替代语法），专门用在"代码和 HTML 混着写"的时候。它比 `<?php } ?>` 好读得多，第 7 篇会细讲。

延伸阅读：[PHP 语法 - 菜鸟教程](https://www.runoob.com/php/php-syntax.html)，[PHP echo/print](https://www.runoob.com/php/php-echo-print.html)，[PHP 字符串](https://www.runoob.com/php/php-string.html)，[PHP 官方手册：字符串](https://www.php.net/manual/zh/language.types.string.php)。

---

> 系列第 2 / 19 篇 · ← 上一步：[[01-PHP 是什么，代码为什么要在服务器上跑|PHP 是什么]] · 下一步：[[03-变量和数据类型|变量和数据类型]] → · 路线图：[[PHP 总览]]
