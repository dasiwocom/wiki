---
title:
aliases:
tags:
  - markdown
description:
draft: false
---
## 行内代码

如果是段落上的一个函数或片段的代码可以用反引号把它包起来

反引号在键盘上的位置是字母Q的左上角

```
使用python输出hello world这句话的代码是`print（"hello world"）`
```

效果：使用python输出hello world这句话的代码是`print（hello world）`

## 代码区块

### 缩进式代码块

代码区块使用4 个空格或者一个制表符Tab 键
和在编辑器里面一样只需按下Tab键缩进然后输入代码就行

    total = 0
    for num in range(1, 6):
        if num % 2 == 0:
            total = total + num
            print(num)
    print("sum =", total)

---
### 三反引号代码块

连续输入三个反引号Obsidian会为你自动生成一个代码框
按下Enter换行开始写入你的代码

````
```
    total = 0
    for num in range(1, 6):
        if num % 2 == 0:
            total = total + num
            print(num)
    print("sum =", total)
```
````

在三反引号后添加语言标识符可以启用语法高亮功能
例如这是一串Python代码，在三反引号后输入python

````
```python
    total = 0
    for num in range(1, 6):
        if num % 2 == 0:
            total = total + num
            print(num)
    print("sum =", total)
```
````    

效果：

```python
    total = 0
    for num in range(1, 6):
        if num % 2 == 0:
            total = total + num
            print(num)
    print("sum =", total)
```

