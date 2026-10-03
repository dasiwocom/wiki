---
title:
aliases:
tags:
  - php
description: 把 PHP 的数组变成 JSON 输出给前端、接住前端发来的 JSON，以及中文字符为什么不能让它转义、空数组为什么变成对象。
---

# JSON，和前端交换数据

## JSON 就是"两边都认识的字符串"

PHP 里数据是数组，JavaScript 里数据是对象，两边不能直接传。中间需要一个共同的格式——JSON。

```text
PHP 数组  →  json_encode  →  JSON 字符串  →  fetch  →  JS 对象
```

PHP 负责编码和解码这两头，中间那段文本谁都能读。

## 把数组变成 JSON

```php
<?php
$user = ["name" => "小明", "age" => 18, "vip" => true];

echo json_encode($user);
```

```text
{"name":"小明","age":18,"vip":true}
```

数字不带引号，`true` 不加引号，字符串带引号——这是 JSON 的规矩，PHP 会自动处理。

但中文默认会被转义：

```php
<?php
echo json_encode(["name" => "小明"]);
```

```text
{"name":"\u5c0f\u660e"}
```

这段东西本身是合法的，前端解析出来也是"小明"，但**你看不懂，调试的时候很难受**。加一个选项：

```php
<?php
echo json_encode($user, JSON_UNESCAPED_UNICODE);
```

```text
{"name":"小明","age":18,"vip":true}
```

几个常用选项：

| 选项 | 作用 |
| --- | --- |
| `JSON_UNESCAPED_UNICODE` | 中文原样输出，不转成 `\uXXXX` |
| `JSON_UNESCAPED_SLASHES` | `/` 不转义成 `\/`（网址看起来正常） |
| `JSON_PRETTY_PRINT` | 加缩进换行，给人看的时候用 |
| `JSON_NUMERIC_CHECK` | 把数字字符串变成数字 |

**写接口基本都带上 `JSON_UNESCAPED_UNICODE`**，这是 PHP 特有的坑，很多语言默认就不会转义。

## 一个必须知道的陷阱：空数组变成 `{}`

```php
<?php
echo json_encode([]);                  // []
echo json_encode(["a", "b"]);          // ["a","b"]

$keyed = ["0" => "a", "2" => "b"];     // 键不连续
echo json_encode($keyed);              // {"0":"a","2":"b"}   —— 变成对象了
```

**PHP 判断"这是列表还是字典"的依据是键：从 0 开始连续整数就是列表，否则是字典。**

键不连续的情况在真实代码里很常见——`unset()` 删了一个元素、`array_filter` 筛过一遍，都会留下空洞：

```php
<?php
$arr = ["a", "b", "c"];
unset($arr[1]);

echo json_encode($arr);
```

```text
{"0":"a","2":"c"}
```

前端拿到这个，`arr.forEach` 会报错，因为它是对象不是数组。

**修法就一句：输出前用 `array_values` 重建索引。**

```php
<?php
echo json_encode(array_values($arr));
```

```text
["a","c"]
```

第 9 篇说过这件事，这里就是它真正会咬人的地方。**凡是数组要输出成 JSON，先想一下它的键是不是连续的。**

反过来，空数组也有讲究：

```php
<?php
// 想让前端收到 []，用这个
echo json_encode(array_values($list));

// 想让前端收到 {}，用这个
echo json_encode((object) $list);
```

`json_encode([])` 给的是 `[]`，一般是对的。但如果你的数据结构是"一串键值对"，前端期望空的时候也是对象，就得强制转成 `(object)`。

## 把 JSON 变成数组

```php
<?php
$json = '{"name":"小明","age":18}';

$arr = json_decode($json, true);    // 第二个参数 true = 转成数组
print_r($arr);
```

```text
Array
(
    [name] => 小明
    [age] => 18
)
```

**第二个参数 `true` 一定要传。** 不传的话得到的是 `stdClass` 对象，取值要写成 `$obj->name` 而不是 `$arr["name"]`，和前面几篇的写法对不上。

```php
<?php
$obj = json_decode($json);           // 不加 true
echo $obj->name;                     // 小明
echo $obj["name"];                   // 报错

$arr = json_decode($json, true);     // 加了 true
echo $arr["name"];                   // 小明
```

解析失败返回 `null`：

```php
<?php
$arr = json_decode($input, true);

if ($arr === null && json_last_error() !== JSON_ERROR_NONE) {
    echo "JSON 格式不对：" . json_last_error_msg();
}
```

**前端传过来的东西不能假设它一定是合法 JSON。** 多一个逗号、少一个引号都会让 `json_decode` 返回 `null`，后面所有取值都会报错。所以要先判断。

## PHP 当接口：输出 JSON

```php
<?php
header("Content-Type: application/json; charset=utf-8");

$users = [
    ["id" => 1, "name" => "小明"],
    ["id" => 2, "name" => "小红"],
];

echo json_encode($users, JSON_UNESCAPED_UNICODE);
```

那行 `header` 是必须的：

- **不加的话**，浏览器按 HTML 处理，前端 `res.json()` 可能解析失败；
- **`charset=utf-8` 不能少**，否则某些环境中文会乱码。

可以在第 11 篇那个 `functions.php` 里封一个函数：

```php
<?php
function jsonOut($data, int $code = 200): void {
    http_response_code($code);
    header("Content-Type: application/json; charset=utf-8");
    echo json_encode($data, JSON_UNESCAPED_UNICODE);
    exit;
}
```

用起来就是一行：

```php
<?php
jsonOut(["ok" => true, "data" => $users]);
jsonOut(["ok" => false, "msg" => "参数不对"], 400);
```

**注意 `exit`。** 输出完就结束，避免后面还有代码又输出一遍，拼出两个 JSON 对象——前端会报"解析失败"，因为那已经不是合法 JSON 了。

统一的返回结构（`ok` + `data` / `msg`）在前端特别好处理，一开始就定下来能省很多事。

## 接住前端发来的 JSON

表单提交是 `application/x-www-form-urlencoded`，`$_POST` 能直接拿到。但用 `fetch` 发的 JSON 请求是另一种格式，`$_POST` 是**空的**，得从原始输入流里读：

```php
<?php
$raw = file_get_contents("php://input");
$data = json_decode($raw, true);

if (!is_array($data)) {
    jsonOut(["ok" => false, "msg" => "请求体不是合法 JSON"], 400);
}

$name = trim($data["name"] ?? "");
```

`php://input` 就是这次请求的原始请求体。**只要前端用 `Content-Type: application/json` 发，就必须用这个方式读。**

## 和前端配合的完整流程

后端 `api/user.php`：

```php
<?php
header("Content-Type: application/json; charset=utf-8");

$users = [
    ["id" => 1, "name" => "小明", "age" => 18],
    ["id" => 2, "name" => "小红", "age" => 22],
];

$keyword = trim($_GET["q"] ?? "");

if ($keyword !== "") {
    $users = array_values(
        array_filter($users, fn($u) => str_contains($u["name"], $keyword))
    );
}

echo json_encode(["ok" => true, "data" => $users], JSON_UNESCAPED_UNICODE);
```

前端页面：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head><meta charset="UTF-8"><title>用户列表</title></head>
<body>
<input type="text" id="q" placeholder="搜索名字">
<button id="btn">搜索</button>
<ul id="list"></ul>

<script>
  document.getElementById("btn").addEventListener("click", async () => {
    const q = document.getElementById("q").value;
    const res = await fetch("api/user.php?q=" + encodeURIComponent(q));
    const json = await res.json();

    const list = document.getElementById("list");
    list.innerHTML = "";

    if (!json.ok) {
      list.textContent = "加载失败";
      return;
    }

    for (const u of json.data) {
      const li = document.createElement("li");
      li.textContent = u.name + "（" + u.age + " 岁）";
      list.appendChild(li);
    }
  });
</script>
</body>
</html>
```

页面上一半是没法在静态页面里演示的（要真有一个 PHP 环境），但这套"`fetch` 请求 → 返回 JSON → 渲染列表"的结构，是现在后台和前端对接的主流做法。前半段在 [[14-异步，等结果回来再往下走|异步]] 那篇讲过，这里补上后端那一半。

几个要注意的地方：

- **前端往网址里拼参数要用 `encodeURIComponent`**，否则搜索词里有 `&`、`#`、空格就会出问题；
- **后端一律用 `array_values` 保证列表键连续**；
- **返回统一带 `ok` 字段**，前端不用靠 HTTP 状态码去猜业务有没有成功；
- **用 `textContent` 而不是 `innerHTML`** 往里写内容，防 XSS（这条和 PHP 端的 `htmlspecialchars` 是同一件事的两半）。

## JSON 和 PHP 数组的对照

| PHP | JSON |
| --- | --- |
| `["a", "b"]` | `["a","b"]` |
| `["name" => "小明"]` | `{"name":"小明"}` |
| `true` / `false` | `true` / `false` |
| `null` | `null` |
| `123` / `1.5` | `123` / `1.5` |
| `[]`（空数组） | `[]` |
| `["k" => null]` | `{"k":null}` |

**PHP 的关联数组、对象，到了 JSON 里都是"对象"。** 这一条把两边的结构对应上了，转换就不会错。

延伸阅读：[PHP JSON - 菜鸟教程](https://www.runoob.com/php/php-json.html)，[PHP 官方手册：json_encode](https://www.php.net/manual/zh/function.json-encode.php)，[JSON 官方说明](https://www.json.org/json-zh.html)，[MDN 的 JSON 介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON)。

---

> 系列第 18 / 19 篇 · ← 上一步：[[17-错误处理和异常，报错怎么查|错误处理和异常]] · 下一步：[[19-连接 MySQL，读写数据|连接 MySQL]] → · 路线图：[[00-PHP 总览]]
