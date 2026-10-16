# WordPress 与 Zibll 源码参考笔记

日期：2026-10-14

## 源码来源

用户提供目录：`C:\Users\windows\Desktop\wordpresszibll`。
WordPress 位于 `www.dasiwo.com`，主题位于 `wp-content/themes/zibll`。主题 style.css 标记版本为 9.1。

本次只静态阅读相关源文件，没有运行站点、导入数据库或修改原始源码。目录中有 SQL 备份，本次没有读取其数据内容。

## 已核实的实现

以下路径相对于 Zibll 主题目录。

- `inc/functions/bbs/inc/class.init.php`：注册版块内容类型 `plate`、帖子内容类型 `forum_post`，以及版块分类 `plate_cat`、帖子话题 `forum_topic`、帖子标签 `forum_tag`。
- `inc/functions/bbs/action/ajax-posts.php`：通过 `wp_insert_post` 保存论坛帖子，正文使用 `post_content`，版块关联存入元数据 `plate_id`，另外保存话题和标签。阅读到的保存流程包含 HTML 编辑内容及隐藏内容处理，不是直接面向 Markdown 原文设计的流程。
- `inc/functions/bbs/inc/home.php`：首页通过 Tab 导航和内容函数组织栏目，并支持快速发帖入口。
- `inc/functions/bbs/inc/posts.php`：`zib_bbs_get_posts_list` 组装列表卡片，包括作者区域、封面、标题、摘要与图片、版块入口及互动信息。`zib_bbs_get_posts_lists_content` 还组合话题、投票和热门评论等内容。
- `inc/functions/bbs/page/home.php`：论坛首页入口调用统一页面模板。

因此需要区分：Zibll 的这些内容共用 WordPress 底层内容设施，但文章、论坛帖子、版块在应用层并非同一个内容类型，也不是只有一套分类。

## 对 Brainpress 第一版的建议（待用户确认）

沿用用户已确认的简单结构：一套文章、单层分类、每篇一个分类、前台投稿、CommonMark 阅读。

列表借鉴论坛卡片的信息层次，先保留：
- 可点击标题。
- 从 Markdown 内容生成的纯文本摘要。
- 分类名称。
- 发布时间。

按发布时间倒序展示是助手建议，尚未确认。作者头像、评论、点赞、收藏、投票、话题、付费权限、独立版块系统不纳入第一版。图片预览、封面和 Tab 切换也不是当前必需项。

本次是源码结构分析，并未实际运行主题查看视觉效果；具体间距、配色和移动端外观仍需后续确认。

## 用户澄清：参考范围

Brainpress 统一使用 post 和文章分类。Zibll 的参考价值集中在前台设计，后续应重点分析页面布局和视觉呈现。其内部内容类型、版块关联等实现细节不影响 Brainpress 已确认的简单内容模型。
