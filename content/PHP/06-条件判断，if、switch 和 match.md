---
title:
aliases:
tags:
  - php
description: if、elseif、switch 和 match 四种分支的写法与区别，以及混写 HTML 时的替代语法。
---

# 条件判断，if、switch 和 match

## 最基础的 if

```php
<?php
$score = 78;

if ($score >= 60) {
    echo "及格了";
}
```

```text
及格了
```

括号里的表达式会被当成真假来判断，规则就是第 3 篇那张"哪些值算假"的表。

## 加分支

```php
<?php
$score = 78;

if ($score >= 90) {
    $level = "优秀";
} elseif ($score >= 75) {
    $level = "良好";
} elseif ($score >= 60) {
    $level = "及格";
} else {
    $level = "不及格";
}

echo $level;
```

```text
良好
```

`elseif` 也可以写成 `else if`，效果一样。**判断是从上往下走的，一旦命中就跳出**，所以顺序很重要——把 `>= 60` 写在最前面，所有人都会变成"及格"。

## 和 HTML 混写时的替代语法

PHP 做网页时，经常要在 HTML 中间插一小段判断。用花括号写出来长这样：

```php
<?php if ($isLogin) { ?>
  <a href="/logout">退出</a>
<?php } else { ?>
  <a href="/login">登录</a>
<?php } ?>
```

能跑，但那几行 `<?php } ?>` 很难看。PHP 提供了替代写法，把花括号换成冒号和 `endif`：

```php
<?php if ($isLogin): ?>
  <a href="/logout">退出</a>
<?php else: ?>
  <a href="/login">登录</a>
<?php endif; ?>
```

读起来清楚多了。**只要代码是"HTML 里夹 PHP"，就一律用替代语法**；纯代码块里还是用花括号。

同样的写法 `for`、`foreach`、`while`、`switch` 都有（`endfor`、`endforeach`、`endwhile`、`endswitch`），第 7 篇会用。

## 三元和 `??`

只有两个结果的时候不用写整个 `if`：

```php
<?php
$age = 18;
$label = $age >= 18 ? "成年" : "未成年";
```

接参数的时候用 `??` 更短：

```php
<?php
$page = (int) ($_GET["page"] ?? 1);
$sort = $_GET["sort"] ?? "new";
```

`??` 只在左边是 `null` 或者不存在时才用右边的值。它和 `?:` 不一样，第 4 篇讲过区别。

## switch：一个值对多个情况

```php
<?php
$action = "edit";

switch ($action) {
    case "new":
        echo "新建";
        break;
    case "edit":
        echo "编辑";
        break;
    case "delete":
        echo "删除";
        break;
    default:
        echo "未知操作";
}
```

```text
编辑
```

三个要注意的地方：

**① `break` 不能漏。** 漏了会"贯穿"到下一个 case 继续执行：

```php
<?php
$n = 1;
switch ($n) {
    case 1:
        echo "一";
    case 2:
        echo "二";
    case 3:
        echo "三";
}
```

```text
一二三
```

全都打出来了——因为 `case 1` 执行完没有 `break`，程序继续往下跑。这个特性偶尔有用（几个 case 共用一段代码），但绝大多数时候是 bug。

**② `switch` 用的是松散比较。** 也就是说 `0 == "abc"` 这类规则在这里生效。PHP 8 之后好一些，但仍然建议：**能用 `match` 就用 `match`**。

**③ 多个 case 共用一段代码时，可以故意不写 `break`：**

```php
<?php
switch ($day) {
    case 6:
    case 7:
        echo "周末";
        break;
    default:
        echo "工作日";
}
```

## match：PHP 8 的新写法

```php
<?php
$action = "edit";

$label = match ($action) {
    "new" => "新建",
    "edit" => "编辑",
    "delete" => "删除",
    default => "未知操作",
};

echo $label;
```

```text
编辑
```

和 `switch` 比，`match` 有几处明显更好：

| | `switch` | `match` |
| --- | --- | --- |
| 比较方式 | 松散比较 `==` | **严格比较 `===`** |
| 是不是表达式 | 不是，只能执行语句 | 是，**能直接赋给变量** |
| 会不会贯穿 | 会，必须写 `break` | 不会 |
| 忘了覆盖所有情况 | 静默往下走 | **抛异常**（`UnhandledMatchError`） |
| 一个分支多值 | 写多个 case | 逗号隔开 |

"能赋值"这一条最实用——不用先声明一个空变量再在分支里赋值了。

多个值匹配一个结果：

```php
<?php
$day = 6;

$type = match ($day) {
    6, 7 => "周末",
    default => "工作日",
};
```

忘了覆盖的后果：

```php
<?php
$n = 5;

$r = match ($n) {
    1 => "一",
    2 => "二",
};
// 报错：UnhandledMatchError: Unhandled match case 5
```

**这叫"宁可报错也不静默出错"**，比 `switch` 那种悄悄什么都不做要安全。

`match` 还能写条件：

```php
<?php
$score = 78;

$level = match (true) {
    $score >= 90 => "优秀",
    $score >= 75 => "良好",
    $score >= 60 => "及格",
    default => "不及格",
};
```

`match (true)` 配合条件表达式，等于把 `if / elseif` 链压缩成一张表，读起来很快。

## 三个该用哪个

- **两三个分支、条件各不相同** → `if`
- **同一个变量和一堆固定值比较、要执行多行代码** → `switch` 或 `match`
- **一个变量对一批固定值、只要一个结果** → `match`

## 嵌套太深的时候

```php
<?php
// 不好读
function check($user) {
    if ($user) {
        if ($user["active"]) {
            if ($user["age"] >= 18) {
                return "通过";
            } else {
                return "年龄不够";
            }
        } else {
            return "账号没激活";
        }
    } else {
        return "没登录";
    }
}
```

条件反向写，把"不满足"的情况提前赶走：

```php
<?php
function check($user) {
    if (!$user) {
        return "没登录";
    }
    if (!$user["active"]) {
        return "账号没激活";
    }
    if ($user["age"] < 18) {
        return "年龄不够";
    }
    return "通过";
}
```

这叫**提前返回**。两层的差别还不明显，四五层的时候一眼就能看出好处——不用再往右边无限缩进了。

## 一个例子

```php
<?php
$hour = (int) date("G");

$period = match (true) {
    $hour < 6  => "凌晨",
    $hour < 12 => "上午",
    $hour < 14 => "中午",
    $hour < 18 => "下午",
    default    => "晚上",
};

echo "现在是{$period}";
```

```text
现在是上午
```

延伸阅读：[PHP If...Else - 菜鸟教程](https://www.runoob.com/php/php-if-else.html)，[PHP Switch](https://www.runoob.com/php/php-switch.html)，[PHP 官方手册：match](https://www.php.net/manual/zh/control-structures.match.php)。

---

> 系列第 6 / 19 篇 · ← 上一步：[[05-常量和字符串，以及常用字符串函数|常量和字符串]] · 下一步：[[07-循环，while、for 和 foreach|循环]] → · 路线图：[[PHP 总览]]
