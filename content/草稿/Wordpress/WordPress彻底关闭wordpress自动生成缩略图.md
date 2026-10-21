---
title:
aliases:
tags:
  - WordPress
description:
draft: false
---
## 一：手动设置关闭

进入WordPress仪表盘(后台)，选择设置

我们在WordPress后台的仪表盘，通过设置→媒体，进入媒体设置页面，如下图所示：

![](https://pic3.zhimg.com/v2-cee6dc3e65dd1267087fb8f83c3c6a76_1440w.jpg)

将你不需要生成的缩略图设置为0即可。

WordPress[上帝模式](https://zhida.zhihu.com/search?content_id=240044845&content_type=Article&match_order=1&q=%E4%B8%8A%E5%B8%9D%E6%A8%A1%E5%BC%8F&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3OTE1NDUwMTYsInEiOiLkuIrluJ3mqKHlvI8iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNDAwNDQ4NDUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.EnTFHmSNESs4Mea3tvgWdlt8h5efqoHm38rhoXlH8uE&zhida_source=entity)

这个时候还没完，**WordPress仍会创建一个固定宽度为768的文件。**

这时候我们就需要开启WordPress上帝模式（全局选项设置）进行设置。

### 进入方法：

访问：http://你的域名/wp-admin/options.php

访问即可进入了WordPress的全部设置（也就是我们常说的WordPress上帝模式）页面。

在此页面找到medium_large_size_w选项，将其设置为 0 即可。记得点保存按钮！

![](https://pic2.zhimg.com/v2-3d7d3281169ba889b910fe8d96a259ed_1440w.jpg)

（建议在该页面使用Ctrl+F进行搜索medium_）