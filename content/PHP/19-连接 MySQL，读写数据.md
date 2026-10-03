---
title:
aliases:
tags:
  - php
description: 用 PDO 连数据库、用预处理语句增删改查、取一行和取多行，以及为什么拼接 SQL 一定会出事。
---

# 连接 MySQL，读写数据

## 先装一个 MySQL

PHP 自己不带数据库。第 1 篇说的 XAMPP、phpStudy 都捆了 MySQL，装完启动就行。

确认 PHP 能连 MySQL：

```php
<?php
var_dump(extension_loaded("pdo_mysql"));
```

返回 `true` 才行。返回 `false` 说明 `php.ini` 里的 `pdo_mysql` 扩展没打开——把前面那行的 `;` 去掉再重启。

## PDO 还是 mysqli

PHP 有两套连 MySQL 的方式：`mysqli` 和 `PDO`。

**用 PDO。** 三个原因：

- 它的**预处理语句**写起来更顺，防注入更自然
- 它也能连 PostgreSQL、SQLite，以后换数据库不用重写
- 出错处理可以通过异常，和其他代码一致

`mysqli` 能用，但没必要二选一的时候折腾。

## 连上去

```php
<?php
$dsn = "mysql:host=localhost;dbname=mysite;charset=utf8mb4";
$pdo = new PDO($dsn, "root", "", [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);
```

三件事值得说：

**① `charset=utf8mb4` 必须写在 DSN 里。** 不写的话，中文可能变成 `???`。`utf8mb4` 比 `utf8` 多了 emoji 的支持，现在一律用它。

**② `ERRMODE_EXCEPTION` 让出错时抛异常。** 默认模式下 PDO 出错是**静默返回 `false`**，你根本不知道出事了。这条设成异常，出错会立刻停下来告诉你原因。

**③ `FETCH_ASSOC` 让查询结果只有字段名做键。** 不设的话，每一行会同时有 `0, 1, 2` 和 `id, name` 两套键，白白多一倍数据。

**把连接写成一个文件**（第 11 篇的 `config.php` + 一个 `db.php`）：

```php
<?php
// db.php
require_once __DIR__ . "/config.php";

function db(): PDO {
    static $pdo = null;         // 一个请求里只连一次

    if ($pdo === null) {
        $dsn = "mysql:host=" . DB_HOST . ";dbname=" . DB_NAME . ";charset=utf8mb4";
        $pdo = new PDO($dsn, DB_USER, DB_PASS, [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]);
    }

    return $pdo;
}
```

用的时候 `db()->prepare(...)` 就行。`static` 那个变量保证一次请求只连一次数据库（第 10 篇讲的 `static` 用法）。

## 建一张表

```sql
CREATE TABLE `message` (
  `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name`       VARCHAR(50)  NOT NULL,
  `content`    TEXT         NOT NULL,
  `created_at` DATETIME     NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

`created_at` 用 `DATETIME` 存"年月日时分秒"的字符串（第 15 篇说过，这个格式能直接排序）。

## 查：取一行

```php
<?php
require_once __DIR__ . "/db.php";

$stmt = db()->prepare("SELECT * FROM message WHERE id = ?");
$stmt->execute([3]);
$row = $stmt->fetch();      // 一行，没有就是 false

if ($row) {
    echo htmlspecialchars($row["name"]) . "：" . htmlspecialchars($row["content"]);
} else {
    echo "这条留言不存在";
}
```

**`prepare` 把 SQL 和变量分开：SQL 里的 `?` 是占位符，值通过 `execute([...])` 传。**

这就是**预处理语句**，它是最重要的两行代码。下面马上说为什么。

## 为什么绝对不能拼 SQL

新手常见的写法：

```php
<?php
// 危险！千万不要这么写
$name = $_POST["name"];
$sql = "SELECT * FROM message WHERE name = '$name'";
$rows = db()->query($sql)->fetchAll();
```

如果用户把名字填成：

```text
' OR '1'='1
```

拼出来的 SQL 就变成：

```sql
SELECT * FROM message WHERE name = '' OR '1'='1'
```

`'1'='1'` 永远成立，**整张表的数据全被查出来了**。如果填的是 `'; DROP TABLE message; --`，表直接被删。

**用预处理语句，占位符里的内容永远被当成"值"，不会被当成 SQL 代码**——这是从机制上解决的，不是靠过滤字符去堵。写了 `prepare` + `execute`，上面那种输入只会被当成一个名字叫 `' OR '1'='1` 的用户去查，查不到而已。

**任何用户输入进 SQL 的地方，一律用占位符。** 没有例外。

## 查：取多行

```php
<?php
$stmt = db()->prepare("SELECT * FROM message ORDER BY id DESC LIMIT ?");
$stmt->bindValue(1, 10, PDO::PARAM_INT);
$stmt->execute();

$rows = $stmt->fetchAll();
```

```text
Array
(
    [0] => Array ( [id] => 5 [name] => 小明 [content] => 第一条 [created_at] => 2026-10-03 14:22:07 )
    [1] => Array ( [id] => 4 [name] => 小红 [content] => 你好 [created_at] => 2026-10-03 14:10:33 )
)
```

**`fetchAll()` 返回二维数组**——刚好就是第 8 篇那种"列表里装字典"，可以直接丢进 `foreach` 渲染成表格。

`LIMIT` 这里要特别注意：它是"结构的一部分"，不能用字符串传：

```php
<?php
// 错：LIMIT '10' 是语法错误
$stmt->execute([10]);

// 对：明确告诉 PDO 这是整数
$stmt->bindValue(1, 10, PDO::PARAM_INT);
```

`bindValue` 的第三个参数指定类型，`PDO::PARAM_INT` 表示按整数处理。

### 用名字占位符更清楚

```php
<?php
$stmt = db()->prepare("SELECT * FROM message WHERE name = :name AND id > :id");
$stmt->execute([":name" => $name, ":id" => $id]);
```

或者不用冒号：

```php
<?php
$stmt->execute(["name" => $name, "id" => $id]);
```

## 插入

```php
<?php
$stmt = db()->prepare(
    "INSERT INTO message (name, content, created_at) VALUES (?, ?, ?)"
);
$stmt->execute([$name, $content, date("Y-m-d H:i:s")]);

$newId = db()->lastInsertId();
echo "新留言的 id 是 " . $newId;
```

`lastInsertId()` 拿到刚插入那行的自增 id——**插入之后要立刻调**，中间再执行别的 SQL 就变成那个的 id 了。

## 更新和删除

```php
<?php
$stmt = db()->prepare("UPDATE message SET content = ? WHERE id = ?");
$stmt->execute([$content, $id]);

echo $stmt->rowCount();     // 影响了几行
```

```php
<?php
$stmt = db()->prepare("DELETE FROM message WHERE id = ?");
$stmt->execute([$id]);

if ($stmt->rowCount() === 0) {
    echo "这条留言本来就不存在";
}
```

**`rowCount()` 用来判断"到底改没改到东西"。** 删除之前先看看有没有删到，比直接报"删除成功"诚实得多。

顺带说一句：留言板这类场合一般**不做物理删除**，而是给表加一个 `deleted_at` 字段，删除时写个时间戳，查询时排除掉。这样误删还能恢复。

## 分页

```php
<?php
$page = max(1, (int) ($_GET["page"] ?? 1));
$size = 10;
$offset = ($page - 1) * $size;

// 总数
$total = db()->query("SELECT COUNT(*) FROM message")->fetchColumn();

// 这一页的数据
$stmt = db()->prepare("SELECT * FROM message ORDER BY id DESC LIMIT ? OFFSET ?");
$stmt->bindValue(1, $size, PDO::PARAM_INT);
$stmt->bindValue(2, $offset, PDO::PARAM_INT);
$stmt->execute();
$rows = $stmt->fetchAll();

$pages = (int) ceil($total / $size);
echo "共 {$total} 条，第 {$page} / {$pages} 页";
```

`OFFSET` 和 `LIMIT` 一样要绑成整数。`fetchColumn()` 直接取第一个字段的值，用来拿 `COUNT(*)` 最方便。

## 一个完整的留言板

```php
<?php
require_once __DIR__ . "/db.php";
date_default_timezone_set("Asia/Shanghai");

$errors = [];
$name = "";
$content = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $name    = trim($_POST["name"] ?? "");
    $content = trim($_POST["content"] ?? "");

    if ($name === "") {
        $errors[] = "名字不能为空";
    }
    if (mb_strlen($content) < 2) {
        $errors[] = "留言至少 2 个字";
    }
    if (mb_strlen($content) > 500) {
        $errors[] = "留言不能超过 500 字";
    }

    if (empty($errors)) {
        try {
            $stmt = db()->prepare(
                "INSERT INTO message (name, content, created_at) VALUES (?, ?, ?)"
            );
            $stmt->execute([$name, $content, date("Y-m-d H:i:s")]);

            header("Location: " . $_SERVER["PHP_SELF"]);
            exit;
        } catch (PDOException $e) {
            error_log($e->getMessage());
            $errors[] = "保存失败，请稍后再试";
        }
    }
}

// 读列表
$rows = db()->query("SELECT * FROM message ORDER BY id DESC LIMIT 20")->fetchAll();
?>
<!DOCTYPE html>
<html lang="zh-CN">
<head><meta charset="UTF-8"><title>留言板</title></head>
<body>
<h1>留言板</h1>

<?php if (!empty($errors)): ?>
  <ul style="color:red">
    <?php foreach ($errors as $e): ?>
      <li><?php echo htmlspecialchars($e); ?></li>
    <?php endforeach; ?>
  </ul>
<?php endif; ?>

<form method="post">
  <p>名字：<input type="text" name="name" value="<?php echo htmlspecialchars($name); ?>"></p>
  <p>留言：<textarea name="content" rows="4"><?php echo htmlspecialchars($content); ?></textarea></p>
  <button type="submit">发表</button>
</form>

<hr>

<?php if (empty($rows)): ?>
  <p>还没有留言。</p>
<?php else: ?>
  <?php foreach ($rows as $r): ?>
    <div>
      <strong><?php echo htmlspecialchars($r["name"]); ?></strong>
      <small><?php echo htmlspecialchars($r["created_at"]); ?></small>
      <p><?php echo nl2br(htmlspecialchars($r["content"])); ?></p>
    </div>
  <?php endforeach; ?>
<?php endif; ?>
</body>
</html>
```

这一段把整组笔记串起来了：

- **第 13 篇的验证**（`trim`、`empty`、长度）
- **第 14 篇的 session**（进阶：让名字自动记住）
- **第 15 篇的日期**（`date` + 时区）
- **第 17 篇的异常处理**（`try/catch` + `error_log`）
- **第 12 篇的表单和回填**
- **预处理语句 + `htmlspecialchars`**

一个能跑的留言板，就是把这几篇拼起来。

## 一个容易忘的细节：`PHP_SELF` 要转义

上面那行 `header("Location: " . $_SERVER["PHP_SELF"])` 在做实际的站里应该这样写：

```php
<?php
header("Location: " . htmlspecialchars($_SERVER["PHP_SELF"]));
```

`PHP_SELF` 里带着网址路径，理论上可以被构造。输出到 HTML 里（比如 `<form action="<?php echo $_SERVER['PHP_SELF']; ?>">`）时**必须转义**。

更简单的做法是直接留空：

```html
<form method="post" action="">
```

提交给当前页面，不用管当前页面地址是什么。

延伸阅读：[PHP MySQL 简介 - 菜鸟教程](https://www.runoob.com/php/php-mysql-intro.html)，[PHP MySQL 连接](https://www.runoob.com/php/php-mysql-connect.html)，[PHP MySQL 插入数据](https://www.runoob.com/php/php-mysql-insert.html)，[PHP 官方手册：PDO](https://www.php.net/manual/zh/book.pdo.php)，[PHP 官方手册：预处理语句与存储过程](https://www.php.net/manual/zh/pdo.prepared-statements.php)。

---

> 系列第 19 / 19 篇 · ← 上一步：[[18-JSON，和前端交换数据|JSON]] · 路线图：[[PHP 总览]]
