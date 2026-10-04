---
title:
aliases:
tags:
  - markdown
description:
draft: false
---
## 绝对路径图片

Markdown图片的语法是

```
![替代文字](图片路径)
```

打开Windows资源管理器的图片右键会出现复制绝对路径的选项，也可以同时按住Ctrl+Shift+C复制

"C:\Users\windows\Pictures\Screenshots\Screenshot 2026-10-03 001833.png"

![[Pasted image 20261004163612.png]]

那么显示这个图片的代码就应该是

```
![屏幕截屏](C:\Users\windows\Pictures\Screenshots\Screenshot 2026-10-03 001833.png)
```

好的我们来看一下

![屏幕截屏](C:\Users\windows\Pictures\Screenshots\Screenshot 2026-10-03 001833.png)

为什么图片无法正常显示？其实我们的语法并没有错
`\`反斜杠在 Markdown 括号内会被当成转义符
`\`在这里是转义起始符号，`\U`、`\P`这些不再被识别成文件夹分隔，路径直接被解析乱了。 关于转义字符，请看[[Markdown转义字符]]
解决方案就是将反斜杠全部变成正斜杠

```
![屏幕截屏](C:/Users/windows/Pictures/Screenshots/Screenshot 2026-10-03 001833.png)
```

展示：

![屏幕截屏](C:/Users/windows/Pictures/Screenshots/Screenshot 2026-10-03 001833.png)

什么情况，竟然还不行，这教程到底靠不靠谱！别急，我们再来学习一下[[URL 与 URL编码]]

看完或许你已经明白反斜杠和空格就是两大病因，修复方法就是用%20代替空格

```
![屏幕截屏](C:/Users/windows/Pictures/Screenshots/Screenshot%202026-10-03%20001833.png)
```

展示：

为了向大家展示我是真的成功了，即便我们使用的是绝对路径
但是当我们换了一个电脑或者环境 这个地址就会失效

![[Pasted image 20261004183135.png]]

或者咱们来到浏览器里copy一个图片地址,URL链接也是一种绝对地址
https://img-blog.csdnimg.cn/84b60bcc127844ddacf39a6d9175af0c.png

![[Pasted image 20261004172637.png]]

```
![屏幕截屏](https://img-blog.csdnimg.cn/84b60bcc127844ddacf39a6d9175af0c.png)
```

![屏幕截屏](https://img-blog.csdnimg.cn/84b60bcc127844ddacf39a6d9175af0c.png)

## 复制粘贴与双链

同时按下Win+Shift+S进入框选截图，截图成功后会自动把截图复制到粘贴板
你也可以去资源管理器或者其他地方直接对图片copy

![[Pasted image 20261004173114.png]]

Ctrl+V粘贴进来

![[Pasted image 20261004173234.png]]
![[Pasted image 20261004173320.png]]

鼠标浮动到图片右上角可以看到他的语法

```
![[Pasted image 20261004173234.png]]
```

粘贴进来的图片Obsidian给他的语法默认使用的是Wikilink，同时会在你指定的目录下保存一份

## 相对目录

上面Obsidian已经替我保存了一份图片在Attachments目录下
Attachments和这篇笔记所在的文件夹Markdown是同级文件夹

相对目录的语法是：

```
![屏幕截图](../Attachments/Pasted%20image%2020261004173114.png)
```

![[Pasted image 20261004183653.png]]

相对目录用起来实在是低效，所以这里不过多阐述，详细请看[[相对路径和绝对路径]]
