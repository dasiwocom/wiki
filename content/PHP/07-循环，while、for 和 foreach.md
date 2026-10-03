---
title:
aliases:
tags:
  - php
description: while、for、foreach 三种循环的写法，break 和 continue 的用法，以及用循环把数组渲染成 HTML 表格。
---

# 循环，while、for 和 foreach

## while：条件成立就一直转

```php
<?php
$i = 1;

while ($i <= 5) {
    echo $i . " ";
    $i++;
}
```

```text
1 2 3 4 5
```

三件事必须想清楚，少一件就会变成死循环：

1. 循环变量从哪开始（上面是 `$i = 1`）
2. 什么条件下继续（`$i <= 5`）
3. 循环变量怎么变（`$i++`）

**忘了第 3 条，页面就会一直转圈。** PHP 有个默认的执行时间上限（通常是 30 秒），到时间会抛一个致命错误，不至于真的卡死电脑，但页面会白屏一段时间。

`while` 最适合"不知道要转几圈"的场合。比如读文件读到结尾：

```php
<?php
$fp = fopen(__DIR__ . "/data.txt", "r");

while (($line = fgets($fp)) !== false) {
    echo htmlspecialchars($line) . "<br>";
}

fclose($fp);
```

一行一行读，读到没有为止——圈数是文件决定的，事先不知道。

## do...while：至少执行一次

```php
<?php
$i = 10;

do {
    echo "执行了一次";
    $i++;
} while ($i < 5);
```

```text
执行了一次
```

条件是 `10 < 5`，假的，但内容已经执行过一次了。`do...while` 的特点就是**先执行、后判断**，至少跑一遍。用得不常见，但写"重试"逻辑时很合适。

## for：知道要转几圈

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    echo $i . " ";
}
```

```text
1 2 3 4 5
```

三个表达式用分号隔开：初始化、条件、每轮之后做什么。最常见的用途是**按索引遍历数组**：

```php
<?php
$arr = ["苹果", "香蕉", "橘子"];

for ($i = 0; $i < count($arr); $i++) {
    echo $arr[$i] . "<br>";
}
```

## foreach：遍历数组专用

这是 PHP 里用得最多的循环，因为你的数据几乎全是数组。

```php
<?php
$fruits = ["苹果", "香蕉", "橘子"];

foreach ($fruits as $fruit) {
    echo $fruit . "<br>";
}
```

```text
苹果
香蕉
橘子
```

要同时拿到键和值：

```php
<?php
$user = ["name" => "小明", "age" => 18, "city" => "杭州"];

foreach ($user as $key => $value) {
    echo "{$key}：{$value}<br>";
}
```

```text
name：小明
age：18
city：杭州
```

**`foreach` 不需要管数组有几个元素，也不会越界。** 只要是要"把数组里每一项都处理一遍"，就一律用 `foreach`，不要用 `for` 配 `count()`。

一个必须知道的细节：foreach 拿到的是**值的副本**。

```php
<?php
$nums = [1, 2, 3];

foreach ($nums as $n) {
    $n = $n * 10;
}

print_r($nums);
```

```text
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
)
```

改了 `$n`，原数组没变。想直接改原数组，要在变量前加 `&`：

```php
<?php
$nums = [1, 2, 3];

foreach ($nums as &$n) {
    $n = $n * 10;
}
unset($n);        // 用完记得断开引用

print_r($nums);
```

```text
Array
(
    [0] => 10
    [1] => 20
    [2] => 30
)
```

**`unset($n)` 这行不能省。** 循环结束后 `$n` 还指着最后一个元素，如果后面还有一个同名的 `foreach`，会把最后那个元素改掉——这是 PHP 里一个著名的隐蔽 bug。

日常建议：**不要用引用，用 `array_map` 生成一个新数组**（第 9 篇），逻辑更清楚，也不会留后患。

## break 和 continue

```php
<?php
for ($i = 1; $i <= 10; $i++) {
    if ($i == 3) {
        continue;      // 跳过这一轮，继续下一轮
    }
    if ($i == 6) {
        break;         // 整个循环到此为止
    }
    echo $i . " ";
}
```

```text
1 2 4 5
```

- **`continue`**：这一轮剩下的代码不执行了，直接进下一轮
- **`break`**：跳出整个循环

用在查找里很常见：

```php
<?php
$found = null;
foreach ($users as $u) {
    if ($u["id"] === 42) {
        $found = $u;
        break;        // 找到了就不用继续找了
    }
}
```

多层嵌套时，`break 2` 表示跳出两层。不过层数一多就该考虑抽函数了。

## 循环里拼 HTML

这是 PHP 循环最实际的用法——把一批数据渲染成表格。

```php
<?php
$goods = [
    ["name" => "键盘", "price" => 199.5],
    ["name" => "鼠标", "price" => 89.0],
    ["name" => "显示器", "price" => 1099.0],
];
?>
<table border="1" cellpadding="6">
  <tr><th>商品</th><th>价格</th></tr>
  <?php foreach ($goods as $g): ?>
    <tr>
      <td><?php echo htmlspecialchars($g["name"]); ?></td>
      <td><?php echo number_format($g["price"], 2); ?></td>
    </tr>
  <?php endforeach; ?>
</table>
```

页面上就是一张三行的表。

这里的 `<?php foreach (...): ?>` 和 `<?php endforeach; ?>` 就是第 6 篇讲的替代语法。**混写 HTML 的循环一律用它**，比 `{` `}` 清楚得多。

数组下标从 0 开始，但表格里常常要从 1 开始编号，可以这样：

```php
<?php foreach ($goods as $i => $g): ?>
  <tr>
    <td><?php echo $i + 1; ?></td>
    <td><?php echo htmlspecialchars($g["name"]); ?></td>
  </tr>
<?php endforeach; ?>
```

`foreach` 的键就是下标，直接拿来做序号。

## 几个常见的死循环

```php
<?php
// ① 忘了让变量变化
$i = 0;
while ($i < 10) {
    echo $i;        // $i 永远是 0
}

// ② 条件永远成立
for ($i = 0; $i >= 0; $i++) { }

// ③ 循环里改了不该改的东西
$arr = [1, 2, 3];
foreach ($arr as $v) {
    $arr[] = $v;    // 边遍历边往数组里加东西
}
```

第三种在 PHP 里其实不会无限跑（`foreach` 遍历的是开始时的副本），但结果通常不是你想要的。**循环里不要去改正在遍历的那个数组**，要改就新建一个。

## 循环该用哪个

| 场景 | 用 |
| --- | --- |
| 遍历数组 | `foreach` |
| 知道要转几次（比如生成 1 到 100） | `for` |
| 不知道几次，边做边判断（读文件、重试） | `while` |
| 至少要做一次 | `do...while` |

**日常写 PHP，`foreach` 占了绝大多数。** 数组操作用 `for` 配 `count()` 是老写法，容易越界，没必要。

## 一个例子

```php
<?php
$scores = ["语文" => 92, "数学" => 78, "英语" => 85];

$sum = 0;
foreach ($scores as $subject => $score) {
    $tag = $score >= 90 ? "优" : ($score >= 80 ? "良" : "中");
    echo "{$subject}：{$score}（{$tag}）<br>";
    $sum += $score;
}

$avg = $sum / count($scores);
echo "平均分：" . round($avg, 1);

// 全部及格就提示
$allPass = true;
foreach ($scores as $score) {
    if ($score < 60) {
        $allPass = false;
        break;
    }
}
echo $allPass ? "，全部及格" : "，有不及格的";
```

```text
语文：92（优）
数学：78（中）
英语：85（良）
平均分：85，全部及格
```

延伸阅读：[PHP While 循环 - 菜鸟教程](https://www.runoob.com/php/php-looping.html)，[PHP For 循环](https://www.runoob.com/php/php-looping-for.html)，[PHP 数组](https://www.runoob.com/php/php-arrays.html)，[PHP 官方手册：流程控制](https://www.php.net/manual/zh/language.control-structures.php)。

---

> 系列第 7 / 19 篇 · ← 上一步：[[06-条件判断，if、switch 和 match|条件判断]] · 下一步：[[08-数组，索引数组、关联数组和遍历|数组]] → · 路线图：[[00-PHP 总览]]
