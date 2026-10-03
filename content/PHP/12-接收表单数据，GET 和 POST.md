---
title:
aliases:
tags:
  - php
description: 表单里的 name 怎么变成 PHP 数组的键、GET 和 POST 该用哪个、复选框和下拉怎么接，以及提交后怎么把用户填的内容留在框里。
---

# 接收表单数据，GET 和 POST

这一篇开始，PHP 真正和浏览器打上交道。前面都是自己算自己的，现在数据从用户那边进来。

## 数据是怎么从表单走到 PHP 的

```mermaid
flowchart LR
  A["浏览器<br/>用户填表单"] --> B["点击提交"]
  B --> C["把 name=值<br/>打包发给服务器"]
  C --> D["PHP 装进<br/>$_GET 或 $_POST"]
  D --> E["你用代码取值"]
```

关键在第三步：**表单里每个控件的 `name` 属性，就是 PHP 数组里的键。**

```html
<form action="hello.php" method="post">
  <input type="text" name="username">
  <button type="submit">提交</button>
</form>
```

用户填 `小明` 提交，PHP 里就是：

```php
<?php
$name = $_POST["username"];    // "小明"
```

`name` 写成 `username`，取值就必须用 `"username"`，**大小写完全一致**。

## `action` 和 `method`

```html
<form action="/save.php" method="post">
```

- **`action`**：提交到哪个文件。留空或写 `""` 表示提交给当前页面自己。
- **`method`**：`get` 还是 `post`，决定数据进 `$_GET` 还是 `$_POST`。

## GET 和 POST，用哪个

| | GET | POST |
| --- | --- | --- |
| 数据放哪 | 网址问号后面 | 请求体里 |
| 看得见吗 | 看得见，会留在地址栏和浏览历史里 | 看不见 |
| 有长度限制吗 | 有（浏览器和服务器都有限制，大约几千字符） | 基本没有 |
| 能存文件吗 | 不能 | 能（第 16 篇） |
| 能收藏/分享链接吗 | 能，链接带着参数 | 不能 |
| 会重复提交吗 | 刷新是安全的 | 刷新会问"是否重新提交" |

选法的规矩很简单：

- **查询、翻页、筛选** → GET。因为结果要能被收藏、被分享、被刷新。
- **新增、修改、删除、登录** → POST。因为改数据的事不该被刷新重做一遍。

**绝对不要用 GET 做删除。** 比如 `/delete.php?id=3`。网址会被浏览器预加载、会被聊天工具抓取预览、会被别人直接访问——都可能导致数据被删掉。

## 取值：先判断再取

```php
<?php
// 方式一：判断存在
if (isset($_GET["id"])) {
    $id = (int) $_GET["id"];
} else {
    $id = 0;
}

// 方式二：?? 更短，等价
$id = (int) ($_GET["id"] ?? 0);
```

**直接写 `$_GET["id"]` 而不判断，参数没传的时候会报"Undefined array key"警告。** 上面两种都行，第二种更常见。

第二个要点是**类型转换**。从网址和表单来的东西**永远是字符串**（或者字符串数组）。要当数字用就得转，否则后面比较大小的时候会出错：

```php
<?php
$page = $_GET["page"] ?? "1";

var_dump($page > 0);          // bool(true)，运气好而已
var_dump($page === 1);        // bool(false) —— 字符串 "1" 不等于数字 1

$page = (int) $page;          // 老老实实转一次
```

## 一个最小的完整例子

```php
<?php
// greeting.php
$name = trim($_POST["username"] ?? "");

if ($_SERVER["REQUEST_METHOD"] === "POST" && $name !== "") {
    echo "<p>你好，" . htmlspecialchars($name) . "</p>";
}
?>
<form method="post" action="">
  <label>你的名字：<input type="text" name="username"></label>
  <button type="submit">提交</button>
</form>
```

`action=""` 表示提交给自己，所以这一个文件同时负责"显示表单"和"处理提交"。用 `$_SERVER["REQUEST_METHOD"]` 区分这两种情况，是最常用的写法。

填上名字点提交，页面顶部就出现"你好，小明"，下面表单还在。

## 各种控件怎么接

### 文本、密码、隐藏域

```html
<input type="text" name="title">
<input type="password" name="pwd">
<input type="hidden" name="id" value="3">
```

```php
<?php
$title = $_POST["title"] ?? "";
$pwd   = $_POST["pwd"] ?? "";
$id    = (int) ($_POST["id"] ?? 0);
```

隐藏域常在"编辑"页面用，把记录的 id 藏在表单里跟着提交回来。

### 单选按钮

```html
<label><input type="radio" name="gender" value="male"> 男</label>
<label><input type="radio" name="gender" value="female"> 女</label>
```

```php
<?php
$gender = $_POST["gender"] ?? "";
```

**同一组单选按钮的 `name` 必须一样**，浏览器保证只有一个会被提交，PHP 收到的是字符串 `"male"`。

这里有个处理上的坑：**要输出回表单时，`value` 和用户选中的值必须用 `===` 比**：

```html
<input type="radio" name="gender" value="male"
  <?php echo ($gender ?? "") === "male" ? "checked" : ""; ?>>
```

用 `==` 比较在这里会出事——如果 `$gender` 是空字符串，`"" == "male"` 是假还好，但某些值组合（比如 `0`）会误判成选中。

### 复选框：不勾选就完全不存在

```html
<label><input type="checkbox" name="agree" value="1"> 同意</label>
```

```php
<?php
$agree = isset($_POST["agree"]);    // 勾了就是 true
```

**没勾选的复选框，浏览器根本不提交它。** 所以这一步不能用 `$_POST["agree"] === "1"`，那个写法在没勾的时候会报"Undefined array key"。用 `isset` 或 `empty`。

一组复选框（多选）用数组语法：

```html
<label><input type="checkbox" name="hobby[]" value="read"> 阅读</label>
<label><input type="checkbox" name="hobby[]" value="music"> 音乐</label>
```

```php
<?php
$hobbies = $_POST["hobby"] ?? [];     // 是一个数组
print_r($hobbies);
```

```text
Array
(
    [0] => read
    [1] => music
)
```

**`name` 后面的 `[]` 是关键**，它告诉 PHP "把同名的值收集成数组"。忘了写，就只有最后一个会留下。

### 下拉框

```html
<select name="city">
  <option value="">请选择</option>
  <option value="hz">杭州</option>
  <option value="bj">北京</option>
</select>
```

```php
<?php
$city = $_POST["city"] ?? "";
```

```html
<option value="hz" <?php echo ($city ?? "") === "hz" ? "selected" : ""; ?>>杭州</option>
```

多选的下拉框加 `multiple`，`name` 要写成 `city[]`。

### 多行文本

```html
<textarea name="content" rows="5"></textarea>
```

```php
<?php
$content = trim($_POST["content"] ?? "");
```

多行文本框里的换行在输出到页面时要处理（第 5 篇的 `nl2br`）：

```php
<?php
echo nl2br(htmlspecialchars($content));
```

**这两个函数要一起用，顺序不能反。** 先 `htmlspecialchars` 把用户写的标签转成文字，再 `nl2br` 补换行。反过来写，`nl2br` 生成的 `<br>` 会被 `htmlspecialchars` 转成文字显示出来。

## 提交后，把用户填的留在框里

表单一旦验证失败，用户得重新填一遍——如果他填的内容全没了，这个体验很差。做法是把上次提交的值填回 `value`：

```html
<form method="post">
  <input type="text" name="title"
         value="<?php echo htmlspecialchars($_POST['title'] ?? ''); ?>">
  <textarea name="content"><?php echo htmlspecialchars($_POST['content'] ?? ''); ?></textarea>
  <button type="submit">保存</button>
</form>
```

三点要注意：

- **`htmlspecialchars` 必须写。** 否则用户输入 `"><script>…` 这种内容，能把你的页面结构撑破，甚至执行脚本。
- 用 `?? ''` 兜底，第一次打开表单时不会报警告。
- `value` 属性用**双引号**，所以里面的 PHP 字符串用**单引号**，避免引号打架。

## 不要用 `$_REQUEST`

```php
<?php
// 看起来很方便，其实是坑
$id = $_REQUEST["id"];
```

`$_REQUEST` 把 GET、POST、Cookie 混在一起，你**没法知道这个值到底是哪来的**。攻击者可以用 Cookie 覆盖掉你期望的表单值。一律明确用 `$_GET` 或 `$_POST`。

## 一个例子：一个简单的搜索框

```php
<?php
$keyword = trim($_GET["q"] ?? "");
$results = [];

if ($keyword !== "") {
    $all = ["PHP 入门", "JavaScript 基础", "CSS 布局", "Linux 常用命令"];

    foreach ($all as $item) {
        if (str_contains($item, $keyword)) {
            $results[] = $item;
        }
    }
}
?>
<form method="get">
  <input type="text" name="q" value="<?php echo htmlspecialchars($keyword); ?>">
  <button type="submit">搜索</button>
</form>

<?php if ($keyword === ""): ?>
  <p>输入关键词开始搜索。</p>
<?php elseif (empty($results)): ?>
  <p>没有找到"<?php echo htmlspecialchars($keyword); ?>"相关的内容。</p>
<?php else: ?>
  <ul>
    <?php foreach ($results as $r): ?>
      <li><?php echo htmlspecialchars($r); ?></li>
    <?php endforeach; ?>
  </ul>
<?php endif; ?>
```

这里用的是 `method="get"`，因为搜索结果要能被收藏、能刷新。输入"CSS"提交之后，地址栏会变成 `?q=CSS`。

延伸阅读：[PHP 表单 - 菜鸟教程](https://www.runoob.com/php/php-forms.html)，[PHP 的 `$_GET` 变量](https://www.runoob.com/php/php-get.html)，[PHP 的 `$_POST` 变量](https://www.runoob.com/php/php-post.html)，[MDN 的表单指南](https://developer.mozilla.org/zh-CN/docs/Learn/Forms)。

---

> 系列第 12 / 19 篇 · ← 上一步：[[11-超级全局变量和 include，把代码拆成几个文件|超级全局变量和 include]] · 下一步：[[13-表单验证，在服务端拦住错误|表单验证]] → · 路线图：[[PHP 总览]]
